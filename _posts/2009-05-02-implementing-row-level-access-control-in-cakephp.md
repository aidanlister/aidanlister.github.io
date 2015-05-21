---
layout: post
title: Implementing row level access control in CakePHP
section: developer
---
The ACL Component in CakePHP is very powerful and can be used to solve a wide variety of access control problems. In this tutorial, we provide a step-by-step guide for implementing row level access control to a model. We will assume at least a basic understanding of ACL and Auth in CakePHP.

The example I am using is an editing platform. At the top of the tree are <code>Volumes</code>, each of which contain many <code>Papers</code>. <code>Authors</code> have access to papers, while <code>Editors</code> have access to volumes and all the papers there-in. Because we have two trees, it's important to design our ACL tree well or it quickly becomes unmanageable.  We'll call the top of our tree <code>Papers</code>, but this choice is arbitrary.

We create the following ACO hierarchy: Papers/<code>Volume</code>/<code>Paper</code>

This tree structure allows you to provide editors access to a volume, which
automatically gives access to the papers inside. This is the beauty of
ACLs.

To create the ACO tree for an existing dataset, we can develop our own cake shell tool. I will assume you have already created the ARO tree for your users and groups.

{% highlight php %}
<?php
class AcltoolShell extends Shell
{
    /**
     * Build a complete ACO tree for two linked models
     *
     * Usage: $ cake acltool aco_models
     */
    function aco_models()
    {
        $this-&gt;out('Starting models sync');
        $Paper  = ClassRegistry::init('Paper');
        $Volume = ClassRegistry::init('Volume');

        // Create the root node
        $root_alias = 'papers';
        $this-&gt;Aco-&gt;create();
        $this-&gt;Aco-&gt;save(array('parent_id' =&gt; null, 'model' =&gt; null, 'alias' =&gt; $root_alias));
        $aco_root = $this-&gt;Aco-&gt;id;

        // Iterate all the volumes
        $volumes = $Volume-&gt;findAll();
        foreach ($volumes as $volume) {
            // Create a node for the volume
            $this-&gt;out(sprintf('Created Aco node: %s/%s', $root_alias, $volume['Volume']['number']));
            $this-&gt;Aco-&gt;create();
            $row = array('parent_id' =&gt; $aco_root, 'foreign_key' =&gt; $volume['Volume']['id'], 'model' =&gt; 'Volume', 'alias' =&gt; $volume['Volume']['number']);
            $this-&gt;Aco-&gt;save($row);
            $parent_id = $this-&gt;Aco-&gt;id;

            // Iterate all the papers
            $papers = $Paper-&gt;find('all', array('conditions' =&gt; array('volume_id' =&gt; $volume['Volume']['id']), 'recursive' =&gt; -1));
            foreach ($papers as $paper) {
                // Create a node for the paper
                $this-&gt;out(sprintf('Created Aco node: %s/%s/%s', $root_alias, $volume['Volume']['number'], $paper['Paper']['slug']));
                $this-&gt;Acl-&gt;Aco-&gt;create();
                $row = array('parent_id' =&gt; $parent_id, 'foreign_key' =&gt; $paper['Paper']['id'], 'model' =&gt; 'Paper', 'alias' =&gt; $paper['Paper']['slug']);
                $this-&gt;Acl-&gt;Aco-&gt;save($row);
            }
        }
    }
}
?>
{% endhighlight %}

Once our ACO tree is built, we need to give our users permissions. Again we will use a cake shell tool.

{% highlight php %}
<?php
    /**
     * Grant user access to two related models
     *
     * Usage: $ cake acltool vol_perms
     */
    function vol_perms()
    {
        // Row level access for volumes
        $this-&gt;out('Creating row-level permissions for volumes');
        $Volume = ClassRegistry::init('Volume');
        $volumes = $Volume-&gt;findAll();
        foreach ($volumes as $vol) {
            $this-&gt;out(sprintf('- Entering volume number %s', $vol['Volume']['number']));
            $Volume-&gt;id = $vol['Volume']['id'];
            foreach ($vol['User'] as $user) {
                $this-&gt;out(sprintf('-- Granting access to %s', $user['name']));
                $User-&gt;id = $user['id'];
                $this-&gt;Acl-&gt;allow($User, $Volume);
            }
        }
    }
}

?&gt;
?>
{% endhighlight %}

We need to inform our models about our chosen ACO structure. We do this for Papers and Volumes in the same way we would for Users and Groups.

{% highlight php %}
<?php
models/volume.php
class Volume extends AppModel
{
    /**
     * Describe our ACO tree
     */
    function parentNode()
    {
        return null;
    }
}
?>
{% endhighlight %}

{% highlight php %}
<?php
models/paper.php
class Paper extends AppModel
{
    /**
     * Describe our ACO tree
     */
    function parentNode()
    {
        if (!$this-&gt;id &amp;&amp; empty($this-&gt;data)) {
            return null;
        }
        $data = $this-&gt;data;
        if (empty($this-&gt;data)) {
            $data = $this-&gt;read();
        }
        if (empty($data['Paper']['volume_id'])) {
            return null;
        } else {
            return array('Volume' =&gt; array('id' =&gt; $data['Paper']['volume_id']));
        }
    }
}
?>
{% endhighlight %}

Unfortunately Auth does not provide a way to automatically handle model access. We use <code>beforeFilter</code> to implement access control manually in the necessary controllers.

First we check that they're not an admin, then we apply our Acl check. This relies on the fact that a) access is blocked to users by the 'controllers' Aco tree and b) access is granted to editors/volumes to this controller by the 'controllers' Aco tree. Both of these constraints are enforced by Auth (with $this->Auth->authorize = 'actions').

{% highlight php %}
<?php
class PapersController extends AppController
{
    /**
     * Row level access checking
     */
    function beforeFilter()
    {
        $methods = array('admin_edit', 'admin_view', 'admin_delete');
        if (isset($this-&gt;params['pass'][0]) &amp;&amp; in_array($this-&gt;params['pass'][0], $methods)) {
            $aco = $this-&gt;Acl-&gt;Aco-&gt;findByModelAndForeignKey('Paper', $this-&gt;params['pass'][0]);
            $aro = $this-&gt;Acl-&gt;Aro-&gt;findByModelAndForeignKey('User', $this-&gt;Auth-&gt;user('id'));
            if (!$this-&gt;Acl-&gt;check($aro['Aro'], $aco['Aco'])) {
                $this-&gt;Session-&gt;setFlash($this-&gt;Auth-&gt;authError);
                $this-&gt;redirect(array('su' =&gt; true, 'controller' =&gt; 'papers', 'action' =&gt; 'index'));
            }
        }
    }
}
?>
{% endhighlight %}

The only thing remaining is displaying a list of papers and volumes
that a user has access to, instead of all the papers/volumes. This is quite difficult for large trees because it's simply not what ACL's are designed for. There's <a href="https://trac.cakephp.org/ticket/6153">work being done</a> in Cake 1.3.x.x to build an access cache which would solve this problem, but in the mean time:

{% highlight php %}
<?php
class PapersController extends AppController
{
    /**
     * Display a list of papers for which the user has access to view
     */
    function admin_index()
    {
        $papers = array();
        $user_id = $this-&gt;Auth-&gt;user('id');
        $nodes = $this-&gt;Acl-&gt;Aro-&gt;findByForeignKeyAndModel($user_id, 'User');
        foreach ($nodes['Aco'] as $node) {
            if ($node['model'] === 'Paper') {
                $papers[] = $node['foreign_key'];
            }

            // Get children from volumes
            if ($node['model'] === 'Volume') {
                $children = $this-&gt;Acl-&gt;Aco-&gt;children($node['id']);
                foreach ($children as $child) {
                    $papers[] = $child['Aco']['foreign_key'];
                }
            }
        }
        $conditions = array('Paper.id' =&gt; $papers);

        if ($this-&gt;Auth-&gt;user('group_id') == 1) {
            $conditions = null;
        }

        $this-&gt;set('papers', $this-&gt;paginate($conditions));
    }
}
?>
{% endhighlight %}

The same applies to the volumes controller, but a little simpler as
you don't need the hierarchy.

{% highlight php %}
<?php
class VolumesController extends AppController
{
    /**
     *
     */
    function admin_index()
    {
        $volumes = array();
        $user_id = $this-&gt;Auth-&gt;user('id');
        $nodes = $this-&gt;Acl-&gt;Aro-&gt;findByForeignKeyAndModel($user_id, 'User');
        foreach ($nodes['Aco'] as $node) {
            if ($node['model'] === 'Volume') {
                $volumes[] = $node['foreign_key'];
            }
        }
        $conditions = array('Volume.id' =&gt; $volumes);
        
        if ($this-&gt;Auth-&gt;user('group_id') == 1) {
            $conditions = null;
        }
        
        $this-&gt;set('volumes', $this-&gt;paginate($conditions));
    }
}
?>
{% endhighlight %}

And that's it, we've implemented complete row-level access control in CakePHP. Cake's AclComponent is incredibly flexible and very powerful, as you can see you can design quite clean and fine-grained permissions quickly and efficiently.

I hope this tutorial helps other developers out there with similar problems, and as always let me know if you have comments or suggestions.