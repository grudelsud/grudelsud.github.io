---
layout: post
title: wordpress authentication - redirect after incorrect login
date: 2011-07-01 15:31:05.000000000 +01:00
categories:
- development
tags:
- PHP
- WordPress
status: publish
type: post
published: true
---
<p>&nbsp;</p>
<div class="zemanta-img zemanta-action-dragged" style="margin: 1em; display: block;">
<p>[caption id="" align="alignright" width="300" caption="Image via Wikipedia"]<a href="http://commons.wikipedia.org/wiki/File:WordPress_logo.svg"><img title="The logo of the blogging software WordPress." src="/images/300px-WordPress_logo.svg.png" alt="The logo of the blogging software WordPress." width="300" height="68" /></a>[/caption]</p>
</div>
<p>Few days ago I had to deploy a simple wordpress installation for a project at @therumpusroom_ with my buddy @twentyrogersc. Super simple, yet it presented a little issue: having a personalized authentication form, displayed on a wordpress page, it was re-directing to the default login screen after specifying incorrect credentials. It came out that we needed to implement the whole authentication mechanism on a page, hence tweaking the simple post form to send data to itself instead of submitting to wp-login.php (which, btw is working fine if users provide the correct credentials, and also accept a redirect parameter to make users land on a pre-determined page).</p>
<p><!--more--></p>
<p>Just for my records, I leave the code here, and I hope I could save some time to people who incur in the same problem. Here's my solution:</p>
<ol>
<li>from the dashboard create a page and give it a meaningful permalink, such as log-in</li>
<li>create a php file and give it a name according to the wordpress convention: page-&lt;permalink&gt;.php (in my case: page-log-in.php)</li>
<li>write the php code to perform the whole authentication (hence including password check and signon), this is my code:
<pre class="brush:php">&lt;?php
    // if user is logged in, redirect whereever you want
    if (is_user_logged_in()) {
        header('Location: '.get_option('siteurl').'/how-to');
        exit;
    }

    // if this page is receiving post data
    // means that someone has submitted the login form
    if( isset( $_POST['log'] ) ) {
        $incorrect_login = TRUE;
        $log = trim( $_POST['log'] );
        $pwd = trim( $_POST['pwd'] );

        // check if username exists
        if ( username_exists( $log ) ) {
            // read user data
            $user_data = get_userdatabylogin( $log );

            // create the wp hasher to add some salt to the md5 hash
            require_once( ABSPATH.'/wp-includes/class-phpass.php');
            $wp_hasher = new PasswordHash( 8, TRUE );
            // check that provided password is correct
            $check_pwd = $wp_hasher-&gt;CheckPassword($pwd, $user_data-&gt;user_pass);

            // if password is username + password are correct
            // signon with wordpress function and redirect wherever you want
            if( $check_pwd ) {
                $credentials = array();
                $credentials['user_login'] = $log;
                $credentials['user_password'] = $pwd;
                $credentials['remember'] = isset($_POST['rememberme']) ? TRUE : FALSE;

                $user_data = wp_signon( $credentials, false );
                header('Location: '.site_url('how-to'));
            }
            else {
                // don't need to do anything here, just print some error message
                // in the form below after checking the variable $incorrect_login
            }
        }
    }

    // and finally print the form, just be aware the action needs to go to "self",
    // hence we're using echo site_url('log-in'); for it
?&gt;
&lt;?php get_header(); ?&gt;

    &lt;h2&gt;log in&lt;/h2&gt;
    &lt;form action="&lt;?php echo site_url('log-in'); ?&gt;" method="post" id="login-form"&gt;
        &lt;label for="log"&gt;User&lt;/label&gt;
        &lt;input type="text" name="log" id="log" class="text" value="&lt;?php echo wp_specialchars(stripslashes($user_login), 1) ?&gt;" size="20" /&gt;

        &lt;label for="pwd"&gt;Password&lt;/label&gt;
        &lt;input type="password" name="pwd" id="pwd" class="text" size="20" /&gt;

        &lt;label for="rememberme"&gt;&lt;input name="rememberme" id="rememberme" type="checkbox" checked="checked" value="forever" /&gt; Remember me&lt;/label&gt;

        &lt;input type="hidden" name="redirect_to" value="&lt;?php echo get_option('siteurl'); ?&gt;/how-to" /&gt;
        &lt;input type="submit" name="submit" value="log in" class="button" /&gt;
    &lt;/form&gt;
&lt;?php
    // incorrect credentials, print an error message
    if( TRUE == $incorrect_login ) {
?&gt;
        &lt;div class="incorrect_login"&gt;Incorrect login details. Please confirm the fields and submit it again,
        or &lt;a href="&lt;?php echo site_url('contact-us'); ?&gt;"&gt;contact us&lt;/a&gt; to obtain a set of credentials.&lt;/div&gt;
&lt;?php
    }
?&gt;   

&lt;?php get_footer(); ?&gt;</pre>
</li>
<li>and finally load this php file where your wordpress theme lives, it's done!</li>
</ol>
