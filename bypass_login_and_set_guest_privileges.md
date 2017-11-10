1. Create a profile (I used - "Guest Profile") for your guests. For a start, I have enabled only the "admin" access rights to the nodes I want guest users to see. 
And to enable user registration and other guest actions, I enabled the "add" access to my user node 
(or on a separate user registration node/table) and other actions on nodes that you may want guests to have access to.
2. Add a user ("Guest Account") account with no username and password to your user table. This will be the initial account of 
anyone who visits your site and has not logged-in yet.
3. Assign the "Guest Profile" to the "Guest Account".
4. Edit config/atk.php to allow multiple authentications including "none". Here, I used the "none" authentication for guest users and 
"db" for registered users. Also, add the "authorization" entry , and set it the same as your second authentication type. 
This prevents ATK from loading "auth_none, db" (as in my authentication) which results in a fatal error because atk does not handle yet 
multiple authorizations.
'''
 /** Security configuration **/
'authentication' => 'none,db',
'authorization' => 'db',
'''
5. Edit **vendor/sintattica/atk/src/Security/SecurityManager.php** to enable correct handling of login out as guest and login in as a 
"member" change :

'''
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
'''

Into:

''' 
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

'''

After this change you will have two option when logging out, calling index.php with the parameter atklogout=true (atklogout=1) 
will log you out, while calling it with atklogout=2 will logout and re-login.

6. Edit **vendor/sintattica/atk/src/Resources/templates/login.tpl** to add a "cancel" button.
''' 
                          <!--
                          <button type="submit" name="login" class="btn btn-primary center-block"
                                  value="{atktext id="login"}">{atktext id="login"}</button>
                          -->
                          <button type="submit" name="login" class="btn btn-primary btn-default"
                                  value="{atktext id="login"}">{atktext id="login"}</button>
 
                          <button type="submit" name="cancel" class="btn btn_cancel btn-default"
                                  value="{atktext id="cancel"}">{atktext id="cancel"}</button>
'''
This will give you a Cancel button that you can click to return to the "home" page without login in.

7. Add a menu entry that allows "Members" to log in. Edit you **src/modules/security/Modules.php** add the following lines to your boot method
'''
$user = \Atk\Securit\SecurityManager::atkGetUser();
          if ($user['name']=='')
          {
              $this->getMenu()->addMenuItem('Miembros','index.php?atklogout=2' , 'main', true, 0, static::$module, '', 'right',true);
          }
'''
8. Add to your language file the translation for "members"
'''
  "members" => "Members",
'''

9.- All set now when you reach the site you'll be presented with the nodes authorized to guest user and if you log in clicking
the "Members" link you'll be presented with the nodes you are authorized.
