---
layout: post
title: Adding a role in drupal
section: developer
---
$roles = user_roles();
$rid = array_search($role, $roles);
$user->roles[$rid] = $role;
user_save($user, array('roles' => $user->roles));
