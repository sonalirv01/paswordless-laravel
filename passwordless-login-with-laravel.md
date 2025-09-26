Passwordless Login with Laravel
An alternative to handling passwords and the processes involved is to drop passwords entirely. Passwordless logins is a login method, in which the user enters his email address only and gets a magic link to sign-in via email. On click of this magic link the sign is processed. This combines password-forgotten, registration and login functionality in one step. It can help to reduce complexity of your web application as well as making it easier for your users. While not everyone is a fan of passwordless sign-in, yet there are a few reasons speaking for it:

Users can't choose insecure passwords or use the same password over all sites, as they don't have a password. We know password managers are the way to go, but let's be realistic: How do you explain the need for a password manager to your 76 year old Mother and explain the password manager to her as well? This is very unlikely to succeed.
Users have to give you a permanent email address - not a ten minute email address. Otherwise they lose access. This might be actually more controversial for some - but what is the point of having users relying on throwaway email addresses?
I've started building a new, secret project recently ;) For this project a passwordless login made most sense as we didn't want to handle all the different steps (register, login, password forgotten, etc.). With passwordless and Laravel it comes down to a few simple methods. Follow the steps below to find out how we've done it. If you get stuck along the way you check out the git repo of the example.

Starting a Fresh Project
For this article, let’s start a fresh Laravel 7 project. You don't actually need a fresh project, it is simply a choice to allow beginners to follow along. It also doesn't require Laravel 7 — earlier versions should support the steps explained as well. Methods might have different names, but the core principals should work the same. Let's get start by creating a new project using composer create-project:

composer create-project --prefer-dist laravel/laravel new-project
cd new-project
After running these two commands you should have a fresh Laravel project and be already in the project folder and ready to continue with the next steps.

Note: Before we continue, a few words more about artisan: Artisan is a command-line tool shipped with Laravel. It allows you to create code and manage your application easier. You don't need to install anything to use it. All you need is your bash command-line open in the project folder. Some of the following commands will start with artisan. If any of these doesn't work for you try php artisan instead.

1. Create the Login View & Route
We start off by creating a view resource to display the login form. In our case, the login form will only consist of one field: the email field. The template is based on MVP.css - a simple CSS library by Andy Brewer. As the name hints, this meant purely as an example. Please don't be disappointed by the rather unfinished pages in this tutorial. Key is the functionality. Designing the form not part of this tutorial.

Create a file under resources/views/login.blade.php with the following contents:

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
    <title>Login</title>
  </head>
  <body>
    <main>
      <section>
        <form>
          <header>
            <h2>Login</h2>
          </header>
          <label for="email">Email Address:</label>
          <input name="email" type="email" size="20" placeholder="Your email address">
          <button type="submit">Login</button>
        </form>
      </section>
    </main>
  </body>
</html>
The second step on this is adding a route so Laravel knows that we plan to display this page when a particular request comes in. Open the routes file for the web app. It is located under routes/web.php and append the following line at the end:

Route::get('/login', function () { return view('login'); });
This tells Laravel to respond to GET-requests for /login by loading the template file from resources/views/login.blade.php, process it and send it to the browser.

Let's test this out and see if it works as expected. If you run artisan serve (or php artisan serve), open your browser, and access your local development environment as printed out by the serve command (usually 127.0.0.1:8000) you should see something like this:

This is the login page, rendered by Firefox. Not pretty - but that's not the point of the tutorial anyways. The styling is left to you: You can opt to style the pages however you like (after getting it working). We will later come back to tweak both files a little more and link them to the functionality we are keen to implement. Let's continue with the second step, implementing a controller to manage the requests.

2. Add the Controller
To add the controller we are using artisan. The commands to create code are called make and followed by a type. The following command creates a controller called AuthController:

artisan make:controller AuthController
After submitting the command you should see "Controller created successfully." in green writing. Open the controller file located under app/Http/Controllers/AuthController.php. You should see the following contents:

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AuthController extends Controller
{
    //
}
 

We will add three methods to handle our login and logout:

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AuthController extends Controller
{
    /**
     * Processes the login form
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function login(Request $request)
    {
        // Coming soon
    }


    /**
     * Processes the actual login
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function signIn(Request $request)
    {
        // Coming soon
    }

    /**
     * Processes the logout
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function logout(Request $request)
    {
        // Coming soon
    }
}
and continue building the functionality later on.

3. Updating the routes
To tell Laravel to use these methods we have to update our routes/web.php and add these new routes to the previously added route:

// Authentication routes
Route::get('/login', function () { return view('login'); });
Route::post('/login', 'AuthController@login')->name('login.submit');
Route::get('/sign-in/{user}', 'AuthController@signIn')->name('sign-in');
Route::get('/logout', 'AuthController@logout')->name('logout');
The whole file should look similar to this now:

<?php

use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});

// Authentication routes
Route::get('/login', function () { return view('login'); });
Route::post('/login', 'AuthController@login')->name('login.submit');
Route::get('/sign-in/{user}', 'AuthController@signIn')->name('sign-in');
Route::get('/logout', 'AuthController@logout')->name('logout');
If you now go back to your browser nothing has changed and the form still doesn't do anything. This is due the reason that we haven't linked the form to the controller yet. We will do this in the following step.

4. Connecting the Login form to the Controller
To connect the form with the controller we need to adjust the login template. Open the file under resources/views/login.blade.php and replace it with this content:

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
    <title>Login</title>
  </head>
  <body>
    <main>
      <section>
        <form action="{{ route('login.submit') }}" method="POST">
          {{ csrf_field() }}
          <header>
            <h2>Login</h2>
          </header>
          <label for="input1">Email Address:</label>
          <input name="email" type="email" size="20" placeholder="Your email address">
          <button type="submit">Login</button>
        </form>
      </section>
    </main>
  </body>
</html>
The new template introduced the following changes:

The form action points now to the route with the name login.submit. This is a so-called "named route".
We have also added another line below the <form>-tag {{ csrf_field() }}: This will ensure that the request can't be easily submitted from elsewhere and increases the security of your form.
After saving the file and reloading the login form in your browser you should be able to submit the form and see a blank page. Why? That's because we aren't returning anything to the browser at this point.

5. Processing the Login Request and Sending the Email.
This is where most of the functionality goes. This will be the meaty part. Next we will add the functionality needed to:

find or create a user account,
an email to this user and
inform the user that the email is on its way.
Let’s break it down into individual steps and have a little conversation around it's workings and the goals.

Find or Create an User Account
With every login request, we get an email address. We need to find out if this email address already exists in our database or we need to create a new user.

Laravel comes with an easy way to find or create an entry in your database. It's called firstOrCreate and takes two arrays as parameters. The first one is the search requirements - this is how your database is searched for an existing entry. The second array contains additional data you want to add in case the entry doesn't exist yet and will be created. In our case this looks like this:

$user = User::firstOrCreate(
    ['email' => $request->post('email')],
    ['name' => $request->post('email'), 'password' => Str::random()]
);
We search by the email as a unique identifier and add name and a randomly generated password in case it will be newly generated. After running this $user will contain a user object for the given email - no matter if it existed before or not.

Prepare URL and send it via email to the user
To allow the user to sign in we have to send an email with a sign-in URL to click on. We start with preparing a temporary and signed URL for the user to click on. This URL is only valid for this particular user and is only valid for 30 minutes:

// create a signed URL for login
$url = URL::temporarySignedRoute(
    'sign-in',
    now()->addMinutes(30),
    ['user' => $user->id]
);
Send an email to the user
Laravel allows you to send simple emails with one button without much configuration. In the following steps we build such an email and send it off to the user. We start by creating a new email using artisan:

artisan make:mail SigninEmail
This creates a new file under app/Mail/SiginEmail.php and prepares the content. We will straight replace it with this:

<?php

namespace App\Mail;

use App\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class SigninEmail extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * @var User
     **/
    protected $user = null;

    /**
     * @var string
     **/
    protected $url = null;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(User $user, $url)
    {
        $this->user = $user;
        $this->url = $url;
    }

 
    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->markdown('mail.signin')
                    ->from('you@yourproject.com')
                    ->to($this->user->email)
                    ->with(['url' => $this->url]);
    }
}
This will load a markdown template called signin from the folder mail. Fill in the placeholders and send the email. Lets create this template file by adding a new file under resources/views/mail/signin.blade.php with the following content:

@component('mail::layout')
{{-- Header --}}
@slot('header')
@component('mail::header', ['url' => config('app.url')])
{{ config('app.name') }}
@endcomponent
@endslot

To sign-in click the button below please:

@component('mail::button', ['url' => $url])
Sign in
@endcomponent

{{-- Footer --}}
@slot('footer')
@component('mail::footer')
© {{ date('Y') }} {{ config('app.name') }}. @lang('All rights reserved.')
@endcomponent
@endslot
@endcomponent
Thanks to Laravels built-in email template, this will prepare a decent looking email out of the box.

If you haven't configured an email sending service, have a look at these mail-sending options. We usually go with Mailgun to avoid ending up in the fifthty percent spam emails.

For development and testing purposes, you can also use mailbase - a local mail catcher tool without any external services. Check the very simple installation instructions on the GitHub repository linked before to get started.

Inform the user the email is sent
Keeping the user in the loop is fairly easy here. We simply display a page stating that we have sent the email and ask to check the users' inbox. Again, to have a simple solution without much styling we are using MVP.css and some basic HTML:

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
    <title>Login</title>
  </head>
  <body>
    <main>
      <h1>Alright!</h1>
      <section>
        <p>Your email is coming shortly. Please check your inbox and click the link in the email!</p>
      </section>
    </main>
  </body>
</html>
We save this into resources/views/login-sent.blade.php and return the rendered view in the controller:

return view('login-sent');
Bringing it together
Putting all of these three steps together into the controller we have the following :

<?php
 
namespace App\Http\Controllers;
 
use App\User;
use App\Mail\SigninEmail;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\URL;
 
class AuthController extends Controller
{
    /**
     * Processes the login form
     *
     * @param \Illuminate\Http\Request $request
     * @return string
     **/
    public function login(Request $request)
    {
        // find the user for the email - or create it.
        $user = User::firstOrCreate(
            ['email' => $request->post('email')],
            ['name' => $request->post('email'), 'password' => Str::random()]
        );
 
        // create a signed URL for login
        $url = URL::temporarySignedRoute(
            'sign-in',
            now()->addMinutes(30),
            ['user' => $user->id]
        );
 
        // send the email
        Mail::send(new SigninEmail($user, $url));
 
        // inform the user
        return view('login-sent');
    }
 
    /**
     * Processes the actual login
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function signIn(Request $request)
    {
        // Coming soon
    }
 
    /**
     * Processes the logout
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function logout(Request $request)
    {
        // Coming soon
    }
}
Please note the updated dependency block on the top:

use App\User;
use App\Mail\SigninEmail;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Mail;
With this the sign-in emails should be sent correctly. An example of an email generated it would look like this:

6. Signing the user in.
Now we have generated a temporary URL and sent it to the user, it is time to process the login request. When we use clicks on the button in the email we got to check the request and process the sign in:

/**
 * Processes the actual login
 *
 * @param \Illuminate\Http\Request $request
 * @param \App\User $user
 * @return redirect
 **/
public function signIn(Request $request, $user)
{
    // Check if the URL is valid
    if (!$request->hasValidSignature()) {
        abort(401);
    }
 
    // Authenticate the user
    $user = User::findOrFail($request->user);
    Auth::login($user);

    // Redirect to homepage
    return redirect('/');
}
After the sign-in has been processed, the user will be redirected to the homepage.

7. Logout
Compared to the sign in steps the sign-out method is pretty trivial. A single call to Auth::logout() ends the session:

/**
 * Processes the logout
 *
 * @param \Illuminate\Http\Request $request
 * @return redirect
 **/
public function logout(Request $request)
{
    // logout
    Auth::logout();
 
    // Redirect to homepage
    return redirect('/');
}
Again, after the action is completed the user will be redirected to the homepage. The final AuthController will look like this:

<?php
 
namespace App\Http\Controllers;
 
use App\User;
use App\Mail\SigninEmail;
use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\URL;
 
class AuthController extends Controller
{
    /**
     * Processes the login form
     *
     * @param \Illuminate\Http\Request $request
     * @return string
     **/
    public function login(Request $request)
    {
        // find the user for the email - or create it.
        $user = User::firstOrCreate(
            ['email' => $request->post('email')],
            ['name' => $request->post('email'), 'password' => Str::random()]
        );
 
        // create a signed URL for login
        $url = URL::temporarySignedRoute(
            'sign-in',
            now()->addMinutes(30),
            ['user' => $user->id]
        );
 
        // send the email
        Mail::send(new SigninEmail($user, $url));
 
        // inform the user
        return view('login-sent');
    }
 
    /**
     * Processes the actual login
     *
     * @param \Illuminate\Http\Request $request
     * @param \App\User $user
     * @return redirect
     **/
    public function signIn(Request $request, $user)
    {
        // Check if the URL is valid
        if (!$request->hasValidSignature()) {
            abort(401);
        }
 
        // Authenticate the user
        $user = User::findOrFail($request->user);
        Auth::login($user);
 
        // Redirect to homepage
        return redirect('/');
    }
 
    /**
     * Processes the logout
     *
     * @param \Illuminate\Http\Request $request
     * @return redirect
     **/
    public function logout(Request $request)
    {
        // logout
        Auth::logout();
 
        // Redirect to homepage
        return redirect('/');
    }
}
That's covering the login and logout functionality. Wherever you are in your Laravel application you can run Auth::user() to access the current user. If it returns null, no one is signed in.

Wrapping It Up!
As you see, a passwordless login is quite easy to implement with Laravel. All core functionality was built with less than 80 lines of code in the controller. With a bit more clean up this can be reduced further. If you think this is too technical for you or you simply need help with a larger project, consider contacting us. We are happy to assist you with different tech stacks and taking on responsibilities from start to go-live and beyond!

If you get stuck during the tutorial in the steps above you might want to consider looking at the example code on GitHub. It contains all steps and the final result, of course.

