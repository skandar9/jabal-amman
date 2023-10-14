Project overview:

Web application designed to cater to the needs of an publication center named Jabal Amman. The project aims to provide a platform that offers various services, products, and resources to residents and visitors of Jabal Amman.

Key Features and Aims:

- *Services:* The project includes a service management system that allows users to browse and access different services available in Jabal Amman. These services could range from everyday necessities, such as utilities and transportation, to leisure and entertainment services.

- *Categories:* The application organizes services and products into categories, making it easier for users to navigate and find specific offerings based on their interests or requirements.

- *Products:* In addition to services, the project incorporates a product catalog that showcases various items available within Jabal Amman. Users can explore and purchase products directly through the platform.

- *Sliders:* The application utilizes image sliders to display visually appealing content.

- *Pictures:* The project allows users to upload and share pictures related to Jabal Amman, fostering a sense of community involvement and visual storytelling.

- *Contact Form:* The project includes a contact form that enables users to get in touch with the administrators or service providers. Users can submit inquiries, feedback, or requests for support, enhancing communication channels within the community.



> :warning: **Warning:** This contents below ↓ contains just parts of my code.
>                        You can access my full project files by clone it from my GitLab repository
>                        (requires asking for my permissions  to grant you access to it):
>                        https://gitlab.com/skandar.s1998/jabal-amman 


## Contents
(contains descriptive parts of my code)

[Tables and relations](#tables-and-relations)

[Project actions and progress(graph)](#project-graph)

[Login to the dashboard functionality](#dashboard-login)

[Create service](#store-service)

[Upload file](#upload-file)

[Products API management (store & show)](#products-api)

["loadImage" JavaScript function (dashboard)](#loadImage)

### **tables-and-relations**

![Logo](/images/tables.png)

For more details about the content of the tables <a href="/jabal-amman.pdf" target="_blank">Click here</a>

[🔝 Back to contents](#contents)

### **project-graph**

This graph diagram represents the actions and progress for the project.

![App Logo](/images/graph.png)

[🔝 Back to contents](#contents)

### **dashboard-login**

![App Logo](/images/login.png)

I used Laravel UI package in this project that provides a convenient way to generate the frontend boilerplate code for authentication views, such as login, registration, and password reset forms.

I Run the following command to install the package:

```shell
composer require laravel/ui
```

This package I used to generate the authentication views.Specifically, I utilized the following file to facilitate the login process for the admin, granting access to their dashboard.

`resources/views/auth/login.blade.php`: This file contains the HTML template for the login form. It includes form fields for the username and password, along with any necessary validation error messages.

I depended on `Auth` Facade that is a core component of Laravel that provides convenient methods for user authentication. It is used to authenticate users and manage their sessions.

### login functionality workflow:

## Routes in `web.php`

`routes\web.php`
### Index Route

```php
Route::get('/', [PagesController::class, 'index'])->name('index');
```
This route is responsible for handling the homepage of the application. When a user visits the root URL, the `index` method of the `PagesController` will be executed.

### Dashboard Route

```php
Route::get('/dashboard', [PagesController::class, 'dashboard'])->middleware(['auth'])->name('dashboard');
```
This route represents the dashboard page of the application. It is protected by the `'auth'` middleware, which means only authenticated users can access it. When a user visits the `/dashboard` URL, the `dashboard` method of the `PagesController` will be executed.


## PagesController

`app\Http\Controllers\PagesController.php`

```php
class PagesController extends Controller
{
    public function index(){
        return redirect('/login');
    }

    public function dashboard(){
        return view('dashboard');
    }
}
```
### `index` Method

This method is responsible for handling the request to the homepage of the application. It performs a redirect to the `/login` URL. When a user visits the homepage, they will be redirected to the login page.

## Authentication Route

`routes\auth.php`

```php
Route::get('/login', [AuthenticatedSessionController::class, 'create'])
                ->middleware('guest')
                ->name('login');

Route::post('/login', [AuthenticatedSessionController::class, 'store'])
                ->middleware('guest');
```
### Login Route
This route is responsible for rendering the login form. When a GET request is made to the `/login` URL, the `create` method of the `AuthenticatedSessionController` will be executed. This route is protected by the `'guest'` middleware, which ensures that only non-authenticated users can access the login page.

### Login POST Route
This route is responsible for handling the submission of the login form. When a POST request is made to the `/login` URL, the `store` method of the `AuthenticatedSessionController` will be executed. This route is also protected by the `'guest'` middleware.

## AuthenticatedSessionController

`app\Http\Controllers\Auth\AuthenticatedSessionController.php`

```php
class AuthenticatedSessionController extends Controller
{
    public function create()
    {
        return view('auth.login');
    }

    public function store(LoginRequest $request)
    {
        $request->authenticate();

        $request->session()->regenerate();

        return redirect()->intended(RouteServiceProvider::HOME);
    }
}
```
### `create` Method
This method is responsible for displaying the login view. It returns the view named `'auth.login'`, which is the login form view. When this method is called, the login form will be rendered and displayed to the user.

### `store` Method
This method handles the incoming authentication request when the login form is submitted. It expects an instance of `LoginRequest` as a parameter, which contains the validation rules for the login request. The method calls the `authenticate` method on the request instance to perform the authentication. If the authentication is successful, the user's session is regenerated, and the user is redirected to the intended page (defined by `RouteServiceProvider::HOME`).

## Login form view

`resources\views\auth\login.blade.php`

```html
<x-layouts.guest>
    <x-auth-card>
        <x-slot name="logo">
            <a href="/">
                <img src="/img/aim.png" width="220px">
            </a>
        </x-slot>

        <!-- Session Status -->
        <x-auth-session-status class="mb-4" :status="session('status')" />

        <!-- Validation Errors -->
        <x-auth-validation-errors class="mb-4" :errors="$errors" />

        <form method="POST" action="{{ route('login') }}">
            @csrf

            <!-- Username -->
            <div>
                <x-label for="username" :value="__('Username')" />
                <x-input id="username" class="block mt-1 w-full" type="text" name="username" :value="old('username')" required autofocus />
            </div>

            <!-- Password -->
            <div class="mt-4">
                <x-label for="password" :value="__('Password')" />
                <x-input id="password" class="block mt-1 w-full"
                                type="password"
                                name="password"
                                required autocomplete="current-password" />
            </div>

            <!-- Remember Me -->
            <div class="block mt-4">
                <label for="remember_me" class="inline-flex items-center">
                    <input id="remember_me" type="checkbox" class="rounded border-gray-300 text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" name="remember">
                    <span class="ml-2 text-sm text-gray-600">{{ __('Remember me') }}</span>
                </label>
            </div>

            <div class="flex items-center justify-end mt-4">
                @if (Route::has('password.request'))
                    <a class="underline text-sm text-gray-600 hover:text-gray-900" href="{{ route('password.request') }}">
                        {{ __('Forgot your password?') }}
                    </a>
                @endif

                <x-button class="ml-3">
                    {{ __('Log in') }}
                </x-button>
            </div>
        </form>
    </x-auth-card>
</x-layouts.guest>
```

The `login.blade.php` file contains the HTML markup for the login form view.
This file is installed with the package `laravel/ui`.

[🔝 Back to contents](#contents)

## LoginRequest

`app\Http\Requests\Auth\LoginRequest.php`

The `LoginRequest` class extends the `FormRequest` class in Laravel and is responsible for handling validation and authentication for the login form submission.
### `authorize` Method
This method determines if the user is authorized to make the login request. In this case, it always returns `true`, allowing any user to make the request.

### `rules` Method
This method defines the validation rules for the login request. It specifies that the `username` and `password` fields are required and must be of type `'string'`.

### `authenticate` Method
This method attempts to authenticate the user using the provided credentials. It first ensures that the request is not rate limited by calling the `ensureIsNotRateLimited` method. Then, it uses the `Auth::attempt` method to attempt authentication using the `username` and `password` fields from the request. It also includes the `remember` value as a boolean parameter to determine if the user wants to be remembered for future sessions. If the authentication attempt fails, the method hits the rate limiter by calling `RateLimiter::hit` and throws a `ValidationException` with the error message `'auth.failed'`. If the authentication attempt succeeds, the rate limiter is cleared by calling `RateLimiter::clear`.

The `LoginRequest` class handles the validation of the login form inputs and performs the authentication logic, including rate limiting to protect against brute-force attacks.

[🔝 Back to contents](#contents)


### **store-service**

`app\Http\Controllers\Dashboard\ServiceController.php`

```php
public function __construct()
{
    $this->middleware('auth');
}
.
.
public function store(Request $request)
{
    $request->validate([
        'name'           => ['required', 'array'],
        'name.ar'        => ['required_with:name'],
        'name.en'        => ['required_with:name'],
        'description'    => ['required', 'array'],
        'description.ar' => ['required_with:description'],
        'description.en' => ['required_with:description'],
        'image'          => ['image','nullable'],
    ]);

    $image = upload_file($request->image,'image_service','services_images');

    Service::create([
        'name'          => $request->name,
        'description'   => $request->description,
        'image'         => $image,
    ]);

    return back()->withStatus(__('service is created successfully'));
}
```

The *__construct* function is called when an instance of the controller is created.
it applies the 'auth' middleware to all methods within the controller. This means that
store function will be protected with this middleware(only auth users can create a new services).

The *store()* function validates the request data, uploads the image and files if present, creates a new publication, and redirects back with a success message.
It called from this route that I defined into *routes\web.php* file:
```php
Route::Resource('/dashboard/services', ServiceController::class)->except(['show']);
```
I used various validation rules, such as ensuring that the 'title' field is an array, the 'title.ar' and 'title.en' fields are required when 'title' is present, the 'image' field is required and should be an image file, and the value of the 'type' field is either 'publications' or 'policies'.

The function then proceeds to upload the image file using the [upload_file function](#upload-file), which takes the 'image' from the request, specifies the file destination directory as 'publications_images', and returns the path of the uploaded image.

[🔝 Back to contents](#contents)

### **upload-file**

`app\helpers.php`

```php
function upload_file($request_file, $prefix, $folder_name)
{
    $extension = $request_file->getClientOriginalExtension();
    $file_to_store = $prefix . '_' . time() . rand(10000, 99999) . '.' . $extension;
    $request_file->storeAs('public/' . $folder_name, $file_to_store);
    return $folder_name . '/' . $file_to_store;
}
```

The `upload_file` function is a utility function that handles file uploads within the application. It performs the following tasks:

1. Takes a file as input and generates a unique filename for storage.
2. Stores the file in the specified folder.
3. Returns the relative path of the uploaded file.

It takes three parameters:

1. `$request_file`: The file obtained from the request.
2. `$prefix`: A prefix string used to create a unique filename.
3. `$folder_name`: The name of the folder where the file will be stored.

The function returns the relative path of the uploaded file. This path is obtained by concatenating the `$folder_name` and the unique filename generated for the file.

[🔝 Back to contents](#contents)

### **products-api**

`routes\api.php`

These two routes define API endpoints related to product management.
- The first route handles retrieving a collection of all products.
- The second route is used to retrieve a specific product by its id.
The assigned names ('products.all' and 'products.show') I used to reference these routes in my application.

```php
Route::get('products', [ProductController::class, 'index'])->name('products.all');
Route::get('products/{product}', [ProductController::class, 'show'])->name('products.show');
```

`app\Http\Controllers\Api\ProductController.php`

```php
class ProductController extends Controller
{
    public function index()
    {
        $products = Product::with('category')->get();
        return ProductResource::collection($products);
    }

    public function show(Product $product)
    {
        return response()->json(new ProductResource($product), 200);
    }
}
```
- `index()` method:
    This method retrieves all products from the database, including their associated categories.
    The retrieved products are then transformed into a collection of [ProductResource](#product-resource) instances using ProductResource::collection($products).

- `show()` method:
    This method retrieves a single product from the database, by expecting a Product model instance to be passed as a parameter.
    The method returns the specified product as a JSON response by creating a [ProductResource](#product-resource) instance for the given $product using new ProductResource($product).

### **#product-resource**

`app\Http\Resources\ProductResource.php`

```php
class ProductResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id'           => $this->id,
            'name'         => $this->name,
            'image'        => $this->image,
            'category_id'  => $this->category_id,
            'capacity'     => $this->capacity,
            'mix_ratio'    => $this->mix_ratio,
            'category'     => new CategoryResource($this->category),
            'translations' => $this->translations,
        ];
    }
}
```
I have created this class to transform the Product model into a customized JSON response with all fields tha written into return statement.

The 'category' field represents the associated category of the product. It is transformed into a CategoryResource instance, which defines transformation logic for the category model that related with Product model (one to many relationship).

[🔝 Back to contents](#contents)

### **loadImage**

![App Logo](/images/loadImage.png)


`resources\views\dashboard\slider_create.blade.php`

```html
<x-layouts.dashboard>
    .
    .
    .
    <x-slot name="head">
        <script>
            var loadImage = function(event) {
              var output = document.getElementById('image');
              output.src = URL.createObjectURL(event.target.files[0]);
              output.style.display = 'inline-block';
                    output.src = URL.createObjectURL(event.target.files[0]);
                    output.onload = function() {
                        URL.revokeObjectURL(output.src)
                    }
            };
        </script>
    </x-slot>
    <form method="post" action="{{ route('sliders.store') }}" autocomplete="off" class="form-horizontal" enctype="multipart/form-data">
        @csrf
        .
        .
        .
        <div class="row">
            <label class="col-sm-3 col-form-label" for="input_image">{{ __('Image') }}</label>
            <div class="col-sm-8">
            <div class="form-group{{ $errors->has('image') ? ' has-danger' : '' }}">
                <input type="file" onchange="loadImage(event)" class="form-control{{ $errors->has('image') ? ' is-invalid' : '' }}" name="image" id="input_image"/>
                <label for="input_image" class="btn btn-primary mt-5 ml-3">Select Image</label>
                @if ($errors->has('image'))
                <span id="image_error" class="error text-danger" for="input_image">{{ $errors->first('image') }}</span>
                @endif
                &nbsp;&nbsp;
                <img id="image" class="rounded" style="height:100px;width:100px;object-fit: cover; display:none">
            </div>
            </div>
        </div>
        <button type="submit"  class="btn btn-success">{{ __('Save') }}</button>
    </form>
    .
    .
    .
</x-layouts.dashboard>
```
The `loadImage` function is responsible for loading and displaying an image selected by the user in an HTML `<input type="file">` element.

1. When the user selects an image file using the file input, the `onchange` event triggers the `loadImage` function.
2. The function receives the event object as a parameter.
3. It starts by retrieving the file input element's value and assigning it to the `output` variable, which represents an `<img>` element that will display the selected image.
4. The `URL.createObjectURL()` method is used to generate a temporary URL for the selected image file. This URL is set as the `src` attribute of the `output` element, which displays the image in the browser.
5. The `output.style.display` property is set to `'inline-block'`, making the image visible.
6. The `URL.createObjectURL()` method is called again to set the `src` attribute of the `output` element. This is done to ensure that the `output.onload` event fires reliably in all browsers.
7. The `output.onload` event is triggered when the image finishes loading. Inside this event handler, the `URL.revokeObjectURL()` method is called to release the temporary URL created by `URL.createObjectURL()`. This helps free up memory resources used by the browser.

[🔝 Back to contents](#contents)