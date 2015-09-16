# RESTful API

This is an new RESTful API for the affiliate4you.

### New End Points

 # Switch to the right API branch
 # Create branch per endpoint/ feature branch
 # Document endpoint in Swagger
 # Write tests for endpoint in the API
 # Write code to expose the endpoint in the API and get green tests (coverage of 95 - 100%)

#### Service Provider

Register the `Mmanos\Api\ApiServiceProvider` in your `app` configuration file.

#### Class Alias

Add a class alias to `app/config/app.php`, within the `aliases` array.

```php
'aliases' => array(
	// ...
	'Api' => 'Mmanos\Api\Api',
)
```

## Laravel 4

Use the `1.0` branch or the `v1.*` tags for Laravel 4 support.

## Configuration

#### Publish config files and migrations

Publish the `lucadegasperi/oauth2-server-laravel` config file and migrations to your application.

```console
$ php artisan vendor:publish --provider="LucaDegasperi\OAuth2Server\OAuth2ServerServiceProvider"
```

Edit the published config file to fit your authentication needs. See this [configuration options](https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Configuration-Options) page for information.

Publish the `mmanos/laravel-api` config file and migrations to your application.

```console
$ php artisan vendor:publish --provider="Mmanos\Api\ApiServiceProvider"
```

And then run the migrations.

```console
$ php artisan migrate
```

Add the following line to your `app/Http/Kernel.php` file in the `$middleware` array to catch any OAuth error and respond appropriately:

```php
'LucaDegasperi\OAuth2Server\Middleware\OAuthExceptionHandlerMiddleware'
```

In order to make some the authorization and resource server work correctly with Laravel 5, remove the `App\Http\Middleware\VerifyCsrfToken` line from the `$middleware` array and place it in the $`routeMiddleware` array like this: `'csrf' => 'App\Http\Middleware\VerifyCsrfToken',`.

> **Note:** remember to add the csrf middleware manually on any route where it's appropriate.

#### Handling Exceptions

We need to modify the exception handler to properly format exceptions thrown by this package. Update the `App/Exceptions/Handler.php` file to use the exception handler from this package.

```php
use Exception;
use Mmanos\Api\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler {
	...
}
```

Then add the `Mmanos\Api\Exceptions\HttpException` exception class to the `$dontReport` array so regular HTTP Exceptions are not reported.

## Controllers

#### Configuration

Add the `ControllerTrait` to each of your API controllers. You could optionally add this to a BaseController extended by all of your other controllers.

```php
use Illuminate\Routing\Controller;
use Mmanos\Api\ControllerTrait;

class BaseController extends Controller
{
	use ControllerTrait;
}
```

#### Pagination

If you return a pagination object from your controller action this package will add the following headers to the response:

* Pagination-Page
* Pagination-Num
* Pagination-Total
* Pagination-Last-Page

#### Setting custom response headers

You may access the response object and set any additional headers directly from your controller action:

```php
$this->response()->header('Example-Header', 'Example value');
```

#### Errors

Dealing with errors when building your API is easy. Simply use the `Api::abort` method to throw an exception that will be formatted in a useful manner.

Throw a 404 Not Found error:

```php
Api::abort(404);
```

Or a 403 Access Denied error:

```php
Api::abort(403);
```

Customize the error message:

```php
Api::abort(403, 'Access denied to scope: users:write');
```

Pass the errors from a validation object to get a clean response with all validation errors:

```php
Api::abort(422, $validator->errors());
```

#### Protecting your API endpoints

You may use the `protect` route filter to ensure the request is authenticated:

```php
$this->beforeFilter('protect');
```

Or you may call the `Api::protect()` method directly.

If this check fails, a call to `Api::abort(401)` is made resulting in an Unauthorized error response.

#### Checking scope access

Use the `checkscope` route filter to ensure the requested resource is accessible:

```php
$this->beforeFilter('checkscope:users.write');
```

Or you may call the `Api::checkScope('users:write')` method directly.

If this check fails, a call to `Api::abort(403)` is made resulting in an Access Denied error response with the scope name.

#### Transforming output

Any model, collection, or pagination object returned by your controller action will be automatically sent through any bound transformer classes.

## Transformers

Transformers allow you to easily and consistently transform objects into an array. By using a transformer you can type-cast integers, type-cast booleans, and nest relationships.

#### Bind a class to a transformer

```php
Api::bindTransformer('User', 'Transformers\User');
```

#### Set a class property

Alternatively, you could add a `transformer` property to your class to be auto-recognized by this package:

```php
class User extends Eloquent
{
	public $transformer = 'Transformers\User';
}
```

#### Creating a transformer class

Ensure your transformer class has a `transform` static method:

```php
namespace Transformers;

class User
{
	public function transform($object, $request)
	{
		$output = $object->toArray();
		$output['id'] = (int) $output['id'];
		$output['created_at'] = $object->created_at->setTimezone('UTC')->toISO8601String();
		$output['updated_at'] = $object->updated_at->setTimezone('UTC')->toISO8601String();
		
		if ($request->input('hide_email')) {
			unset($output['email']);
		}
		
		return $output;
	}
}
```

## Internal Requests

A big part of this package is being able to perform requests on your API internally. This allows you to build your application on top of a consumable API.

#### Performing requests

Use the `Api::internal()` method to initiate an internal request:

```php
$users_array = Api::internal('api/users')->get();
```

#### Passing extra parameters

```php
$users_array = Api::internal('api/users', array('sort' => 'newest'))->get();
```

#### Specify HTTP method

```php
$new_user_array = Api::internal('api/users', array('email' => 'test@example.com'))->post();
```

## CORS Support

CORS support is enabled by default, but only if the `Origin` header is detected. Adjust the settings in the config file to control the behavior and header values.
