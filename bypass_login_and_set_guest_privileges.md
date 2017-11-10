## Bypass authorization and set guest privileges

### A word of caution.

This technique involves modifying atk source code! So, after applying it you should "freeze" the requirements with composer as an update
will most likely "destroy" the fix rendering your site buggy and/or unusable.


### Allright, let's do it.

1. Create a profile (I used - "Guest Profile") for your guests. For a start, I have enabled only the "admin" access rights to the nodes I want guest users to see. 
And to enable user registration and other guest actions, I enabled the "add" access to my user node 
(or on a separate user registration node/table) and other actions on nodes that you may want guests to have access to.
2. Add a user ("Guest Account") account with no username and password to your user table. This will be the initial account of 
anyone who visits your site and has not logged-in yet.
3. Assign the "Guest Profile" to the "Guest Account".
4. Edit **config/atk.php** to allow multiple authentications including "none". Here, I used the "none" authentication for guest users and 
"db" for registered users. Also, add the "authorization" entry , and set it the same as your second authentication type. 
This prevents ATK from loading "auth_none, db" (as in my authentication) which results in a fatal error because atk does not handle yet 
multiple authorizations.

```php

 /** Security configuration **/
'authentication' => 'none,db',
'authorization' => 'db',

```

5. Edit **vendor/sintattica/atk/src/Security/SecurityManager.php** to enable correct handling of login out as guest and login in as a 
"member", change :

```php

public function run()
    {
        global $ATK_VARS;
        $isCli = php_sapi_name() === 'cli';
        // Logout?
        if (isset($ATK_VARS['atklogout'])) {
            $this->logout();
            if (!$isCli) {
                header('Location: '.Config::getGlobal('dispatcher'));
            }
            exit;
        }
        // Get some vars
        $session = &SessionManager::getSession();
        $auth_rememberme = isset($ATK_VARS['auth_rememberme']) ? $ATK_VARS['auth_rememberme'] : 0;
        if (Config::getGlobal('auth_loginform') == true) {
            // form login
            $auth_user = isset($ATK_VARS['auth_user']) ? $ATK_VARS['auth_user'] : '';
            $auth_pw = isset($ATK_VARS['auth_pw']) ? $ATK_VARS['auth_pw'] : '';
        } else {
            // HTTP login
            $auth_user = isset($_SERVER['PHP_AUTH_USER']) ? $_SERVER['PHP_AUTH_USER'] : '';
            $auth_pw = isset($_SERVER['PHP_AUTH_PW']) ? $_SERVER['PHP_AUTH_PW'] : '';
        }
        // try a session login
        if (isset($session['login']) && $session['login'] == 1) {
            $this->sessionLogin();
        }
   
        ...
```

Into:

```php

     public function run()
     {
         global $ATK_VARS;

         $isCli = php_sapi_name() === 'cli';

         // Get some vars
         $session = &SessionManager::getSession();
         $auth_rememberme = isset($ATK_VARS['auth_rememberme']) ? $ATK_VARS['auth_rememberme'] : 0;

         if (Config::getGlobal('auth_loginform') == true) {
             // form login
             $auth_user = isset($ATK_VARS['auth_user']) ? $ATK_VARS['auth_user'] : '';
             $auth_pw = isset($ATK_VARS['auth_pw']) ? $ATK_VARS['auth_pw'] : '';
         } else {
             // HTTP login
             $auth_user = isset($_SERVER['PHP_AUTH_USER']) ? $_SERVER['PHP_AUTH_USER'] : '';
             $auth_pw = isset($_SERVER['PHP_AUTH_PW']) ? $_SERVER['PHP_AUTH_PW'] : '';
         }

         // Logout?
         if (isset($ATK_VARS['atklogout'])) {
             $this->logout();
             //Do Re-Login
             if ($ATK_VARS['atklogout'] == 2)
             {
                 $this->loginForm($auth_user, $this->m_fatalError);
             }
             if (!$isCli) {
                 header('Location: '.Config::getGlobal('dispatcher'));
             }
             exit;
         }

         // try a session login
         if (isset($session['login']) && $session['login'] == 1) {
             $this->sessionLogin();
         }
   
         ...
```

After this change you will have two option when logging out, calling index.php with the parameter atklogout=true (atklogout=1) 
will log you out, while calling it with atklogout=2 will logout and re-login.

6. Edit **vendor/sintattica/atk/src/Resources/templates/login.tpl** to add a "cancel" button if in "guest mode" and you want
to abort login in.

```smarty
     {if $atklogout == 2}
              <button type="submit" name="login" class="btn btn-primary btn-default"
              value="{atktext id="login"}">{atktext id="login"}</button>

              <button type="submit" name="cancel" class="btn btn_cancel btn-default"
              value="{atktext id="cancel"}">{atktext id="cancel"}</button>
      {else}
              <button type="submit" name="login" class="btn btn-primary center-block"
              value="{atktext id="login"}">{atktext id="login"}</button>

      {/if}

```
This will give you a Cancel button that you can click to return to the "home" page without login in.
Now we need to pass the variable **atklogout** to the template engine, we'll need to modify **vendor/sintattica/atk/src/Security/SecurityManager.php** again to pass the variable, change the "loginForm" method to:

```php 
    public function loginForm($defaultname, $error = '')
    {
         global $ATK_VARS;
         $page = Page::getInstance();
         $ui = Ui::getInstance();

         $page->register_script(Config::getGlobal('assets_url').'javascript/tools.js');

         $tplvars = [];
         $tplvars['atksessionformvars'] = Tools::makeHiddenPostvars(['atklogout', 'auth_rememberme', 'u2f_response']);
         $tplvars['atklogout'] = $ATK_VARS['atklogout'];
         $tplvars['formurl'] = Config::getGlobal('dispatcher');
         $tplvars['username'] = Tools::atktext('username');

         ...
```
This way, if you change your mind about bypassing login, you can change **config/atk.php** to disable authentication none and your login
box will show only the login button again.

7. Add a menu entry that allows "Members" to log in. Edit you **src/modules/security/Modules.php** add the following lines to your boot method

```php

   $user = \Sintattica\Atk\Security\SecurityManager::atkGetUser();
   if ($user['name']=='')
   {
       $this->getMenu()->addMenuItem('Miembros','index.php?atklogout=2' , 'main', true, 0, static::$module, '', 'right',true);
   }

```
In a matter of fact, you can add this lines to **any** Module.php (i.e. being in the security/Modules.php is not mandatory) as they are
only there to make a menu item appear on the menu bar, it just happen that adding it to security module makes sense.

8. Add to your language file the translation for "members"

```php

"members" => "Members",

```

9.- All set now when you reach the site you'll be presented with the nodes authorized to guest user and if you log in clicking
the "Members" link you'll be presented with the nodes that you are authorized to.
