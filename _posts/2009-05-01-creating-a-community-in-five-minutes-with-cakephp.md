---
layout: post
title: Creating a community in five minutes with CakePHP
section: developer
summary: |
    CakePHP's automatic hashing makes things a lot harder than they need to be, and simple tasks (e.g. a registration page) become annoyingly difficult.
    
    Here, we build a complete community based website in five minutes using Cake best practices, with the following features:
    <ul>
        <li>Account registration</li>
        <li>Login and logout</li>
        <li>Account management page</li>
        <li>Password retrieval</li>
    </ul>
---
CakePHP's automatic hashing makes things a lot harder than they need to be, and simple tasks (e.g. a registration page) become annoyingly difficult.

<strong>Update</strong>: Now compatible with CakePHP 1.3.x. You can also <a href="http://github.com/aidanlister/cakecommunity">download the source code directly</a> from GitHub.

Here, we build a complete community based website in five minutes using Cake best practices, with the following features: account registration, login and logout, account management page, and password retrieval.

We will take the following steps:
<ol>
    <li>Bake a new Cake website</li>
    <li>Build an account registration page</li>
    <li>Create a login page, and optionally add a <code>lastlogin</code> field</li>
    <li>Add change password functionality to our account page</li>
    <li>Finish with the password retrieval feature</li>
</ol>

At any time you can <a href='/assets/img/uploads/2009/05/building-a-community-in-cakephp.zip'>download the source code</a> used in this tutorial. Let's get started!

<h3>Step 1. Bake a new Cake website</h3>

I'm going to assume that you have a working <a href="http://book.cakephp.org/view/108/The-CakePHP-Console">cake shell installation</a>, so change to your DocumentRoot and execute:
<code>$ cake bake community</code>

This will create a <code>community</code> folder with all the necessary files for your new website. If you visit http://example.com/community, you should see the "all green" indicating you have a working cake installation.

Next, we'll create the <code>users</code> table by executing the following SQL:

{% highlight php %}
<?php
CREATE TABLE `users` (
  `id` int(11) NOT NULL auto_increment,
  `created` datetime,
  `modified` datetime,
  `lastlogin` datetime,
  `name` varchar(100),
  `email` varchar(200),
  `username` varchar(50),
  `password` varchar(42),
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`)
)
?>
{% endhighlight %}

That's it for the setting up ... Let's move on.

<h3>Step 2. Build an account registration page</h3>

We'll get straight into it by creating the users controller, model and the register view.

Because of CakePHP's automatic hashing, we have to do all our validation work on the <code>password_confirm</code> field, as the <code>password</code> field contains only the hash of the users password.

{% highlight php %}
<?php
class User extends AppModel
{
    /**
     * Standard validation behaviour
     */
    var $validate = array(
        'name' => array(
            'length' => array(
                'rule'      => array('minLength', 5),
                'message'   => 'Please enter your full name (more than 5 chars)',
                'required'  => true,
            ),
        ),
        'username' => array(
            'length' => array(
                'rule'      => array('minLength', 5),
                'message'   => 'Must be more than 5 characters',
                'required'  => true,
            ),
            'alphanum' => array(
                'rule'      => 'alphanumeric',
                'message'   => 'May only contain letters and numbers',
            ),
            'unique' => array(
                'rule'      => 'isUnique',
                'message'   => 'Already taken',
            ),
        ),
        'email' => array(
            'email' => array(
                'rule'      => 'email',
                'message'   => 'Must be a valid email address',
            ),
            'unique' => array(
                'rule'      => 'isUnique',
                'message'   => 'Already taken',
            ),
        ),
        'password' => array(
            'empty' => array(
                'rule'      => 'notEmpty',
                'message'   => 'Must not be blank',
                'required'  => true,
            ),
        ),
        'password_confirm' => array(
            'compare'    => array(
                'rule'      => array('password_match', 'password', true),
                'message'   => 'The password you entered does not match',
                'required'  => true,
            ),
            'length' => array(
                'rule'      => array('between', 6, 20),
                'message'   => 'Use between 6 and 20 characters',
            ),
            'empty' => array(
                'rule'      => 'notEmpty',
                'message'   => 'Must not be blank',
            ),
        ),
    );
    

    /**
     * Ensure two password fields match
     *
     * @param   array   data provided by the controller
     * @param   string  the original (already hashed) password fieldname
     * @param   bool    whether the password field has been hashed,
     *                  only hashed when a username input is present
     */
    function password_match($data, $password_field, $hashed = true)
    {        
        $password         = $this->data[$this->alias][$password_field];
        $keys             = array_keys($data);
        $password_confirm = $hashed ?
              Security::hash($data[$keys[0]], null, true) :
              $data[$keys[0]];
        return $password === $password_confirm;
    }
}
?>
{% endhighlight %}

Next we create our view at <code>views/users/register.ctp</code>. Because we've done all our validation on the <code>password_confirm</code> field, the error messages (password too short, etc.) will appear under <code>password_confirm</code>. This isn't ideal from the user interface perspective; luckily we can just swap the input labels and noone is the wiser.

{% highlight php %}
<?php
views/users/register.ctp:
<?php
echo $this->Form->create(array('action' => 'register'));
echo $this->Form->input('name');
echo $this->Form->input('email');
echo $this->Form->input('username');
echo $this->Form->input('password_confirm', array('label' => 'Password', 'type' => 'password'));
echo $this->Form->input('password', array('label' => 'Password Confirm'));
echo $this->Form->end('Register');
?>
{% endhighlight %}

Finally we create our users controller at <code>controllers/users_controller.php</code>. We define a <code>beforeFilter</code> method to make our registration page publicly accessible, our <code>register</code> action, and a blank <code>login</code> action.

{% highlight php %}
<?php
class UsersController extends AppController
{
    var $components = array('Auth');
    
    /**
     * Runs automatically before the controller action is called
     */
    function beforeFilter()
    {
        $this->Auth->allow('register');
        parent::beforeFilter();
    }
    
    /**
     * Registration page for new users
     */
    function register()
    {
        if (!empty($this->data)) {
            $this->User->create();
            if ($this->User->save($this->data)) {
                $this->Session->setFlash(__('Your account has been created.', true));
                $this->redirect('/');
            } else {
                $this->Session->setFlash(__('Your account could not be created.', true));
            }
        }
    }
    
    /**
     * Account login page
     */
    function login()
    {
    
    }

    /**
     * Log a user out
     */
    function logout()
    {
       return $this->redirect($this->Auth->logout());
    }
}
?>
{% endhighlight %}

The next issue with Auth arises when the user has entered a valid password, but has failed to validate the entire form - for example, if they picked a username that is in use and must be re-entered. As the password has already been replaced by the hashed password, the form will be redisplayed with the hashed password in the <code>password</code> input. If the user attempts to resubmit the form without retyping their password, form validation will fail on the <code>password_confirm</code> field. Even worse, if you're not using a <code>password_confirm</code> field, the user will simply be unable to log in to their newly created account.

Thus, we need some hackery to make sure encrypted passwords are not sent back to the user. We can do this in <code>AppController</code>, by writing a <code>beforeRender</code> method.

{% highlight php %}
<?php
app_controller.php:
class AppController extends Controller
{
    /**
     * Before Render
     */
    function beforeRender()
    {
        unset($this->data['User']['password']);
        unset($this->data['User']['password_confirm']);
    }
}
?>
{% endhighlight %}

And that's it, we have a complete working registration page. We can view this completed page by visiting <code>/users/register</code> and create our first account.

<h3>Step 3. Create a login page</h3>

Next up is our login page. We need to modify the default layout which lives at <code>views/layouts/default.ctp</code>. We replace the <code>content</code> div which allows <code>Auth</code> to display messages to the user.

{% highlight php %}
<?php
views/layouts/default.ctp:
<div id="content">
    <?php echo $this->Session->flash(); ?>
    <?php echo $this->Session->flash('auth'); ?>
    <?php echo $content_for_layout; ?>
</div>
?>
{% endhighlight %}

Next we create a very simple login page:

{% highlight php %}
<?php
views/users/login.ctp:
<?php
echo $this->Form->create(array('action' => 'login'));
echo $this->Form->input('username');
echo $this->Form->input('password');
echo $this->Form->end('Login');
?>
{% endhighlight %}

And that's it - it doesn't get much simpler. We can view our new login page at <code>/users/login</code>.

If we wanted to track the time the user logged in last, we can do some extra work:

Adding to the <code>AppController</code>, we set <code>Auth->autoRedirect</code>:

{% highlight php %}
<?php
app_controller.php:
class AppController extends Controller
{
    ...

    /**
     * Before Filter
     */
    function beforeFilter()
    {
        $this->Auth->autoRedirect = false;
    }
}
?>
{% endhighlight %}

Then we modify our login controller to do the necessary work:

{% highlight php %}
<?php
controllers/users_controller.php:
class UsersController extends AppController
{
    ...

    /**
     * Ran directly after the Auth component has executed
     */
    function login()
    {
        // Check for a successful login
        if (!empty($this->data) && $id = $this->Auth->user('id')) {

            // Set the lastlogin time
            $fields = array('lastlogin' => date('Y-m-d H:i:s'), 'modified' => false);
            $this->User->id = $id;
            $this->User->save($fields, false, array('lastlogin'));

            // Redirect the user
            $url = array('controller' => 'users', 'action' => 'account');
            if ($this->Session->check('Auth.redirect')) {
                $url = $this->Session->read('Auth.redirect');
            }
            $this->redirect($url);
        }
    }

}
?>
{% endhighlight %}

Not a lot of additional work, and it will come in handy if you need to prune inactive accounts. Note we can't this part of the step yet, as it's expecting an <code>/users/account</code> page to exist (which we create next).

<h3>Step 4. Add change password functionality</h3>

Cake's form validation functionality is somewhat limited, in that it only allows you to define validation rules at the model level. Often, you will want to validate data at a per-form level instead. To achieve this, we can dynamically modify the validation rules before form validation takes place. We dive into the user model again:

Note: If we simply added the <code>password_old</code> rule to our validation behaviour, other forms that didn't include <code>password_old</code> would not validate (because we have set <code>required = true</code>). If we removed <code>required = true</code>, a POST request can easily be forged bypassing this security check.

{% highlight php %}
<?php
models/user.php:
class User extends AppModel
{
    ...

    /**
     * Extra form dependent validation rules
     */
    var $validateChangePassword = array(
        '_import' => array('password', 'password_confirm'),
        'password_old' => array(
            'correct' => array(
                'rule'      => 'password_old',
                'message'   => 'Does not match',
                'required'  => true,
            ),
            'empty' => array(
                'rule'      => 'notEmpty',
                'message'   => 'Must not be blank',
            ),
        ),
    );

    /**
     * Dynamically adjust our validation behaviour
     *
     * Look for an _import key in new ruleset, and import
     * those rules from the base validation ruleset
     *
     * @param   string  array key of the validation ruleset to use
     */
    function useValidationRules($key)
    {
        $variable = 'validate' . $key;
        $rules = $this->$variable;
        
        if (isset($rules['_import'])) {
            foreach ($rules['_import'] as $key) {
                $rules[$key] = $this->validate[$key];
            }
            unset($rules['_import']);
        }
        
        $this->validate = $rules;
    }
    

    /**
     * Ensure password matches the user session
     *
     * @param   array   data provided by the controller
     */
    function password_old($data)
    {
        $password = $this->field('password',
            array('User.id' => $this->id));
        return $password ===
            Security::hash($data['password_old'], null, true);
    }

    ...
}
?>
{% endhighlight %}

Our change password logic now becomes very simple. We validate, then update the password and redirect the user.

Note: In this case Cake decides not to automatically hash the users password field, so we must perform the hashing manually. This happens because Cake will only hash the password if both the <code>username</code> and <code>password</code> key are set.

{% highlight php %}
<?php
controllers/users_controller.php:
class UsersController extends AppController
{
    /**
     * Account details page (change password)
     */
    function account()
    {
        // Set User's ID in model which is needed for validation
        $this->User->id = $this->Auth->user('id');
        
        // Load the user (avoid populating $this->data)
        $current_user = $this->User->findById($this->User->id);
        $this->set('current_user', $current_user);

        $this->User->useValidationRules('ChangePassword');
        $this->User->validate['password_confirm']['compare']['rule'] =
            array('password_match', 'password', false);

        $this->User->set($this->data);
        if (!empty($this->data) && $this->User->validates()) {
            $password = $this->Auth->password($this->data['User']['password']);
            $this->User->saveField('password', $password);

            $this->Session->setFlash('Your password has been updated');
            $this->redirect(array('action' => 'account'));
        }        
    }

    ...
}
?>
{% endhighlight %}

We create a template for the account page as follows:
{% highlight php %}
<?php
views/users/account.ctp:
<h2>Account Page</h2>
<h3>Change your password</h3>
<p>You are <?php echo $current_user['User']['name']; ?> who last logged in <?php echo $current_user['User']['lastlogin']; ?>.</p>

<?php
echo $this->Form->create(array('action' => 'account'));
echo $this->Form->input('password_old',     array('label' => 'Old password', 'type' => 'password', 'autocomplete' => 'off'));
echo $this->Form->input('password_confirm', array('label' => 'New password', 'type' => 'password', 'autocomplete' => 'off'));
echo $this->Form->input('password',         array('label' => 'Re-enter new password', 'type' => 'password', 'autocomplete' => 'off'));
echo $this->Form->end('Update Password');
?>
?>
{% endhighlight %}

The user is now able to change their password. We can test this by logging in at <code>/users/login</code> which will take us to our new account page.

<h3>Step 5. Create a password retrieval page</h3>

In our final step, we allow the user to retrieve a forgotten password. We do this by emailing the user a token. The user clicks back to our website with the token, and gets emailed a replacement password.

We create a <code>models/token.php</code> to handle the token generation, information storage and retrieval.

{% highlight php %}
<?php
models/token.php:
class Token extends AppModel 
{
    /**
     * Create a new ticket by providing the data to be stored in the ticket. 
     */
    function generate($data = null)
    {
        $data = array(
          'token' => substr(md5(uniqid(rand(), 1)), 0, 10),
          'data'  => serialize($data),
        );
        
        if ($this->save($data)) {
            return $data['token'];
        }
        
        return false;
    }
    
    /**
     * Return the value stored or false if the ticket can not be found.
     */
    function get($token)
    {
        $this->garbage();
        $token = $this->findByToken($token);
        if ($token) {
          $this->delete($token['Token']['id']);
          return unserialize($token['Token']['data']);
        }
          
        return false;
    }
  
    /**
     * Remove old tickets
     */
    function garbage()
    {   
        return $this->deleteAll(array('created < INTERVAL -1 DAY + NOW()'));
    }
}
?>
{% endhighlight %}

We'll also create the necessary database table.

<code lang="sql">
CREATE TABLE `tokens` (
  `id` int(11) unsigned NOT NULL auto_increment,
  `created` datetime default NULL,
  `modified` datetime default NULL,
  `token` varchar(32) default NULL,
  `data` text,
  PRIMARY KEY  (`id`),
  UNIQUE KEY `token` (`token`)
)
</code>

We next update the users controller to make the <code>recover</code> and <code>verify</code> actions public, then we define our <code>recover</code> and <code>verify</code> actions:

{% highlight php %}
<?php
controllers/users_controller.php:
<?php
class UsersController extends AppController
{
    var $components = array('Auth', 'Email');
    
    ...
    
    /**
     * Runs automatically before the controller action is called
     */
    function beforeFilter()
    {
        $this->Auth->allow('register', 'recover', 'verify');
        parent::beforeFilter();
    }
    
    /**
     * Allows the user to email themselves a password redemption token
     */
    function recover()
    {
        if ($this->Auth->user()) {
            $this->redirect(array('controller' => 'users', 'action' => 'account'));
        }
        
        if (!empty($this->data['User']['email'])) {
            $Token = ClassRegistry::init('Token');
            $user = $this->User->findByEmail($this->data['User']['email']);
            
            if ($user === false) {
                $this->Session->setFlash('No matching user found');
                return false;
            }
            
            $token = $Token->generate(array('User' => $user['User']));
            $this->Session->setFlash('An email has been sent to your account, please follow the instructions in this email.');
            $this->Email->to = $user['User']['email']; 
            $this->Email->subject = 'Password Recovery'; 
            $this->Email->from = 'Support <support@example.com>';
            $this->Email->template = 'recover';
            $this->set('user', $user);
            $this->set('token', $token);
            $this->Email->send();
        }
    }
    
    /**
     * Accepts a valid token and resets the users password
     */
    function verify($token_str = null)
    {
        if ($this->Auth->user()) {
            $this->redirect(array('controller' => 'users', 'action' => 'account'));
        }

        $Token = ClassRegistry::init('Token');
        
        $res = $Token->get($token_str);
        if ($res) {
            // Update the users password
            $password = $this->User->generatePassword();
            $this->User->id = $res['User']['id'];
            $this->User->saveField('password', $this->Auth->password($password));
            $this->set('success', true);

            // Send email with new password
            $this->Email->to = $res['User']['email'];
            $this->Email->subject = 'Password Changed';
            $this->Email->from = 'Support <support@example.com>';
            $this->Email->template = 'verify';
            $this->set('user', $res);
            $this->set('password', $password);
            $this->Email->send();
        }
    }

}
?>
{% endhighlight %}

Because we are generating new passwords for users, we need a function to do this. Alternative approaches, like <a href="http://pear.php.net/package/Text_Password">PEAR's Text/Password</a> can be substituted if you like.

{% highlight php %}
<?php
models/user.php:
class User extends AppModel
{
    ....

    /**
     * Generate a random pronounceable password
     */
    function generatePassword($length = 10) {
        // Seed
        srand((double) microtime()*1000000);
        
        $vowels = array('a', 'e', 'i', 'o', 'u');
        $cons = array('b', 'c', 'd', 'g', 'h', 'j', 'k', 'l', 'm', 'n',
            'p', 'r', 's', 't', 'u', 'v', 'w', 'tr',
            'cr', 'br', 'fr', 'th', 'dr', 'ch', 'ph',
            'wr', 'st', 'sp', 'sw', 'pr', 'sl', 'cl');
        
        $num_vowels = count($vowels);
        $num_cons = count($cons);
        
        $password = '';
        for ($i = 0; $i < $length; $i++){
            $password .= $cons[rand(0, $num_cons - 1)] . $vowels[rand(0, $num_vowels - 1)];
        }
        
        return substr($password, 0, $length);
    }
?>
{% endhighlight %}

We define two email templates for recovery and verification respectively:

{% highlight php %}
<?php
views/elements/email/text/recover.ctp:
Dear <?php echo $user['User']['name']; ?>,

Someone is attempting to reset your password.

Your username for this account is: <?php echo $user['User']['username']; ?>


If you wish to continue, you may reset your password by
following this link:

    <?php echo Router::url(array('controller' => 'users', 'action' => 'verify', $token), true); ?>


If you did not initiate this action, please contact
support. You can log in to change your password
at this address:

    <?php echo Router::url(array('controller' => 'users', 'action' => 'login'), true); ?>
    

Thanks,
Support
?>
{% endhighlight %}

{% highlight php %}
<?php
views/elements/email/text/verify.ctp:
Dear <?php echo $user['User']['name']; ?>,

Your password has been reset, please use the following
details to log into our site.

    Username: <?php echo $user['User']['username']; ?>
    
    Password: <?php echo $password; ?>


Please change your password to something more memorable.
You can log in to change your password at this address:

    <?php echo Router::url(array('controller' => 'users', 'action' => 'login'), true); ?>
    

Thanks,
Support
?>
{% endhighlight %}

And finally we define our two templates for the recovery and verification views:

{% highlight php %}
<?php
views/users/recover.ctp:
<h2>Recover Password</h2>

<?php
echo $this->Form->create('User', array('action' => 'recover'));
echo $this->Form->input('email');
echo $this->Form->end('Recover');
?>
?>
{% endhighlight %}

{% highlight php %}
<?php
views/users/verify.ctp:
<h2>Recover Password</h2>

<?php if (isset($success)): ?>
    <div class="message">Access verified. Your new password has been emailed to you.</div>
    <p>A new password has been generated for your account and mailed to you. After you've logged in, you should change your password to something memorable via the account information page.</p>
<?php else: ?>
    <div class="warning">Invalid token. This page has expired, or the link was not copied from your email client correctly.</div>
    <p>Make sure you have copied the entire link correctly, pasting it together if the link was split over two lines. If you're copying the link correctly and still can't get access, please contact us.</p>
<?php endif; ?>
?>
{% endhighlight %}

And that's it ... we've developed a complete community orientated website for CakePHP in less time it takes than to make a cup of tea (if you're very slow at making tea).

On a side note: you may have noticed that CakePHP's automatic hashing makes things a lot harder than they should be. In my opinion, automatic hashing directly goes against Cake's mantra of "If it's easy to do in Cake, then it doesn't belong in core", and I hope the core development team rethink in the idea in Cake 1.3.x.x. Although forcing developers to automatically hash their users password is a nice idea, the current implementation a major stumbling block for new bakers. We shouldn't need a <a href="http://book.cakephp.org/view/565/Troubleshooting-Auth-Problems ">section in the manual</a> about common problems the user will encounter trying to do simple tasks. Instead, we should fix the cause of the common problems. It is, however, a testament to Cake's flexibility that we are able to work around most of these issues quite elegantly. It's also worth noting that there are several ways to <a href="http://teknoid.wordpress.com/2008/10/08/demystifying-auth-features-in-cakephp-12/">override this behavior</a>.

I hope you've enjoyed this tutorial, please feel free to leave comments, suggestions or improvements.