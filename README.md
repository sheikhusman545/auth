## Gelembjuk/Auth package

PHP Package for easy login to a web site with popular social networks.

Now supports login with Facebook, Google, Twitter, LinkedIn, Xing.

### Installation

Using composer: [gelembjuk/auth](http://packagist.org/packages/gelembjuk/auth) ``` require: {"gelembjuk/auth": "*"} ```

### Configuration

You need to get API keys for each of social networks.

Details on how to register application and get keys for each social network.

**Twitter** Register your application there https://apps.twitter.com/ . Copy Consumer Key and Secret Key in your application settings.

**LinkedIn** Go to https://developer.linkedin.com/, Then "My Apps". Create new App. Copy API Key and Secret. You need to set Authorized Redirect URLs. 
It will be an url to your "Login complete script".

**Facebook** Go to https://developers.facebook.com/. "Add new App" as Web-Page. Copy API Key and Secret. Enable "Web OAuth Login".

**Google** Go to https://code.google.com/apis/console. Create new Project. Enable Google+ API in this project in the APIs section. Then Create new Client ID in Credentials section.
Choose "Web Application". Set up redirect url , it will be an url to your "Login complete script". Copy Client ID , Client Secret

**NOTE**. For some social networks it is needed to provide a correct login redirect url 
(callback url) in an application settings when you registering new keys. For example, you have to do this is Google API Console.

Login redirect url is a "complete login" url in your app. A script where you do final actions on a social login.

Recommended is to keep all integration options in one array in a separate safe configuration file. It is not required to have all integrations keys. Add only what you really need.

```php
$integrations = array(
	'facebook' => array(
		'api_key' => 'fake facebook api key',
		'secret_key' => 'fake facebook secret key'
		),
	'twitter' => array(
		'consumer_key' => 'fake twitter consumer key',
		'consumer_secret' => 'fake twitter consumer secret'
		),
	'linkedin' => array(
		'api_key' => 'fake linkedin api key',
		'api_secret' => 'fake linkedin api secret'
		),
	'google' => array(
		'application_name' => 'Your application name',
		'client_id' => 'fake google api client id',
		'client_secret' => 'fake google api client secret'
		),
	'xingapi' => array(
		'consumer_key' => 'fake xing consumer key',
		'consumer_secret' => 'fake xing counsumer secret'
		),
	'liveid' => array(
		'consumer_key' => 'fake live id consumer key',
		'consumer_secret' => 'fake live id consumer secret'
		)
	);

```

### Usage

#### Start login process. 

File startlogin.php

```php

// composer autoloader
require '../vendor/autoload.php';

$socialnetwork = $_REQUEST['network'];  // this is one of: facebook, google, twitter, linkedin

// create social network login object. The second argument is array of API settings for a social network
$network = Gelembjuk\Auth\AuthFactory::getSocialLoginObject($socialnetwork,$integrations[$socialnetwork]);

$redirecturl = 'http://'.$_SERVER['HTTP_HOST'].dirname($_SERVER['REQUEST_URI']).'/completelogin.php';

$socialauthurl = $network->getLoginStartUrl($redirecturl);
		
// remember the state. it will be used when complete a social login
$_SESSION['socialloginsate_'.$socialnetwork] = $network->serialize();

// this is optional. you can include a network name in your redirect url and then extract
$_SESSION['socialloginnetwork'] = $socialnetwork;

header("Location: $socialauthurl",true,301);
exit;

```

#### Complete login process.

File completelogin.php

```php

// composer autoloader
require '../vendor/autoload.php';

$socialnetwork = $_SESSION['socialloginnetwork']; 

$network = Gelembjuk\Auth\AuthFactory::getSocialLoginObject($socialnetwork,$integrations[$socialnetwork]);

try {
	// read some input parameters needed to complete auth by this social network
	$arguments = array();
	
	foreach ($network->getFinalExtraInputs() as $key) {
		$arguments[$key] = $_REQUEST[$key];
	}
	
	// restore to a state before redirect
	$network->unSerialize($_SESSION['socialloginsate_'.$socialnetwork]);
			
	// get authorized user short social profile
	$profile = $network->completeLogin($arguments);
	
	// save user info to a session
	$_SESSION['user'] = $profile;
	
	// now user is loged in!
	
	//redirect user to home page
	header("Location: index.php",true,301);
	exit;
	
} catch (Exception $e) {
	echo "Somethign went wrong during the login process<br>";
	echo "Error is: ".$e->getMessage();
}

```

### Author

Roman Gelembjuk (@gelembjuk)

