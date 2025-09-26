
# Passwordless Login with Laravel

Passwordless login is a modern authentication method where users sign in using a magic link sent to their email, eliminating the need for passwords. This approach simplifies registration, login, and password recovery into a single step, improving user experience and security.

---

## Why Passwordless?

- **No insecure passwords:** Users can't reuse or choose weak passwords.
- **Permanent email required:** Users must use a real email, not a disposable one.
- **Simplified onboarding:** Registration and login are merged.

---

## Getting Started

### 1. Create a New Laravel Project

```bash
composer create-project --prefer-dist laravel/laravel new-project
cd new-project
```

---

### 2. Create the Login View

Create `resources/views/login.blade.php`:

```html
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
          <label for="email">Email Address:</label>
          <input name="email" type="email" size="20" placeholder="Your email address">
          <button type="submit">Login</button>
        </form>
      </section>
    </main>
  </body>
</html>
```

---

### 3. Add Routes

Edit `routes/web.php`:

```php
use Illuminate\Support\Facades\Route;

Route::get('/login', function () { return view('login'); });
Route::post('/login', 'AuthController@login')->name('login.submit');
Route::get('/sign-in/{user}', 'AuthController@signIn')->name('sign-in');
Route::get('/logout', 'AuthController@logout')->name('logout');
```

---

### 4. Create the Controller

Generate the controller:

```bash
artisan make:controller AuthController
```

Edit `app/Http/Controllers/AuthController.php`:

```php
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
    public function login(Request $request)
    {
        $user = User::firstOrCreate(
            ['email' => $request->post('email')],
            ['name' => $request->post('email'), 'password' => Str::random()]
        );

        $url = URL::temporarySignedRoute(
            'sign-in',
            now()->addMinutes(30),
            ['user' => $user->id]
        );

        Mail::send(new SigninEmail($user, $url));

        return view('login-sent');
    }

    public function signIn(Request $request, $user)
    {
        if (!$request->hasValidSignature()) {
            abort(401);
        }

        $user = User::findOrFail($request->user);
        Auth::login($user);

        return redirect('/');
    }

    public function logout(Request $request)
    {
        Auth::logout();
        return redirect('/');
    }
}
```

---

### 5. Create the Mail Class

Generate the mail class:

```bash
artisan make:mail SigninEmail
```

Edit `app/Mail/SigninEmail.php`:

```php
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

    protected $user;
    protected $url;

    public function __construct(User $user, $url)
    {
        $this->user = $user;
        $this->url = $url;
    }

    public function build()
    {
        return $this->markdown('mail.signin')
                    ->from('you@yourproject.com')
                    ->to($this->user->email)
                    ->with(['url' => $this->url]);
    }
}
```

---

### 6. Create the Email Markdown Template

Create `resources/views/mail/signin.blade.php`:

```blade
@component('mail::layout')
{{-- Header --}}
@slot('header')
@component('mail::header', ['url' => config('app.url')])
{{ config('app.name') }}
@endcomponent
@endslot

{{-- Body --}}
To sign-in, click the button below:

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
```

---

### 7. Inform the User

Create `resources/views/login-sent.blade.php`:

```html
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
```

---

## Final Thoughts

- Passwordless login is easy to implement in Laravel.
- All core logic fits in a single controller.
- For more details, see the [example code on GitHub](https://github.com/your-repo).

---

## Useful Links

- [Laravel Documentation](https://laravel.com/docs)
- [Mailgun](https://www.mailgun.com/)
- [Mailbase (local mail catcher)](https://github.com/mailbase/mailbase)
- [MVP.css](https://andybrewer.github.io/mvp/)

---

**Happy coding!**
