# Laravel Passport
**Laravel Passport** is an **OAuth2 server and API authentication package** for the Laravel framework. Passport is built on top of the League OAuth2 server that is maintained by Andy Millington and Simon Hamp.
https://github.com/thephpleague/oauth2-server
## Installation and setup
`php artisan install:api --passport`
This command will publish and run the database migrations for creating the tables needs to store OAuth2 clients and access tokens.

Additionally, the command will ask, like to use UUIDs as the primary key value of the Passport Client model instead of auto-incrementing integers. Press yes if UUIDs need to use.
After API scaffolding installed. Then add the **Laravel\Passport\HasApiTokens** trait to the User model.
### User Model
	namespace App\Models;
	use Illuminate\Database\Eloquent\Factories\HasFactory;
	use Illuminate\Foundation\Auth\User as Authenticatable;
	use Illuminate\Notifications\Notifiable;
	use Laravel\Passport\HasApiTokens;
	class User extends Authenticatable
	 {
		 use HasApiTokens, HasFactory, Notifiable;
	 }

Finally, in the application's **config/auth.php** configuration file, define an api authentication guard and set the driver option to passport.

	'guards' => [
		   'web' => [
				  'driver' => 'session',
				  'provider' => 'users',
			],
		   'api' => [
				  'driver' => 'passport',
				  'provider' => 'users',
			 ],
	  ],

## Protecting Routes via Middleware
Passport includes an authentication guard that will validate access tokens on incoming requests. The api guard to use the passport driver, need to specify the **auth:api **middleware on any routes that should require a valid access token.

	Route::get('/user', function () {
		//business logic
	})->middleware('auth:api');

## Practical Implementation

Create new controller inside API directory:
php artisan make:controller
**name** : API\AuthController
**type**:  API
**model**: User

Add following **routes** inside **api.php**

	use App\Http\Controllers\api\AuthController;
	Route::post('/register', [AuthController::class,'register'] );
	Route::post('/login', [AuthController::class,'login'] );
	Route::get('/me', [AuthController::class,'me'])->middleware('auth:api');
	Route::delete('/logout', [AuthController::class,'logout'])->middleware('auth:api');

Create following functions inside **AuthController**:

	use Illuminate\Http\Response;
	public function register(Request $request): Response
	 {
		  $user=new User($request->all());
		  $user->save();
		  return Response(['status' => 200,'data' => $user],200);
	 }

	public function login(Request $request): Response
	 {
		   $input = $request->all();
		   Auth::attempt($input);
		   $user = Auth::user();
		   $token = $user->createToken('example')->accessToken;
		   return Response(['status' => 200,'token' => $token],200);
	 }

	public function me(): Response
	 {
		 if(Auth::guard('api')->check()){
				$user = Auth::guard('api')->user();
				return Response(['data' => $user],200);
		  } 
		  return Response(['data' => 'Unauthorized'],401);
	 }
	 
	 public function logout(): Response
	 {
		   if(Auth::guard('api')->check()){
				 $accessToken = Auth::guard('api')->user()->token();
				\DB::table('oauth_refresh_tokens')
					 ->where('access_token_id', $accessToken->id)
					 ->update(['revoked' => true]);
				  $accessToken->revoke();
				 return Response(['data' => 'Unauthorized','message' => 'User logout successfully.'],200);
			  }
			  return Response(['data' => 'Unauthorized'],401);
	 }

