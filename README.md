"Jabal Amman" is a web application designed to cater to the needs of a specific location or community named Jabal Amman. The project aims to provide a platform that offers various services, products, and resources to residents and visitors of Jabal Amman. It serves as a centralized hub for information and interaction within the community.


Apologies for any confusion. Based on the 29 files you have provided, I will now provide a description of the project "Jabal Amman" and its aims:

"Jabal Amman" is a comprehensive web application designed to cater to the needs of a specific location or community named Jabal Amman. The project aims to provide a platform that offers various services, products, and resources to residents and visitors of Jabal Amman. It serves as a centralized hub for information and interaction within the community.

Key Features and Aims:

1. Services: The project includes a service management system that allows users to browse and access different services available in Jabal Amman. These services could range from everyday necessities, such as utilities and transportation, to leisure and entertainment services.

2. Categories: The application organizes services and products into categories, making it easier for users to navigate and find specific offerings based on their interests or requirements.

3. Products: In addition to services, the project incorporates a product catalog that showcases various items available within Jabal Amman. Users can explore and purchase products directly through the platform.

4. Sliders: The application utilizes image sliders to display visually appealing content, such as promotional banners, event highlights, or community announcements. Sliders serve as an effective means of engaging users and conveying essential information.

5. Pictures: The project allows users to upload and share pictures related to Jabal Amman, fostering a sense of community involvement and visual storytelling.

6. Contact Form: The web application includes a contact form that enables users to get in touch with the administrators or service providers. Users can submit inquiries, feedback, or requests for support, enhancing communication channels within the community.

By providing a centralized platform for services, products, and community interaction, "Jabal Amman" aims to achieve the following benefits:

1. Convenience: Residents and visitors of Jabal Amman can access a wide range of services and products from a single platform, simplifying their daily activities and enhancing convenience.

2. Community Engagement: The project fosters a sense of community engagement by allowing users to interact through features like picture sharing, contact forms, and discussions on various topics.

3. Local Economy Boost: By promoting local businesses and services, the project plays a vital role in supporting the local economy of Jabal Amman, encouraging growth and sustainability.

4. Information Hub: "Jabal Amman" serves as an information hub, providing users with up-to-date details on services, products, events, and community news, ensuring they stay informed and connected.

5. Seamless User Experience: The application aims to deliver a seamless and user-friendly experience, making it easy for users to navigate, explore, and engage with the available resources.

Overall, "Jabal Amman" is a dynamic web application that strives to enhance the quality of life in the Jabal Amman community. By offering a comprehensive platform for services, products, and community interaction, the project aims to provide convenience, foster engagement, boost the local economy, and serve as an information hub for residents and visitors alike.
## Contents

[Graph](#graph)

[Authentication](#authentication)

[Store service function](#store-service)

[Upload file function](#upload-file)

[Products API management (store & show)](#products-api)

[Ckeditor (text editor) functionality (dashboard)](#Ckeditor)

["loadImage" JavaScript function (dashboard)](#loadImage)

### **authentication**
![App Logo](/images/graph.png)

[🔝 Back to contents](#contents)

### **authentication**

app\Http\Controllers\AuthController.php:

The constructor, this function is part of a controller class and is responsible for setting up themiddleware for authentication using Laravel Sanctum.
This middleware ensures that the user is authenticated using the Sanctum authentication guardbefore accessing the methods.
The $this->middleware('auth:sanctum')->only(['logout', 'user']); line specifies that the'auth:sanctum' middleware should be applied only to the 'logout' and 'user' methods.

```php
    public function __construct()
    {
        $this->middleware('auth:api')->only(['logout']);
    }
```

```php
    public function register (Request $request) 
    {
        $request->validate([
            'name'       => ['required', 'string'],
            'email'      => ['required', 'string', 'email', 'unique:users'],  
            'password'   => ['required', 'string', 'min:6', 'confirmed'],
        ]);

        $user = User::create([
            'name'      => $request->name,
            'email'     => $request->email,
            'password'  => Hash::make($request->password),
            'role'      => 'user',
            'balance'   => 0,
        ]);

        $token = $user->createToken('Proxy App')->accessToken;
        
        return response()->json([
            'user' => new UserResource($user),
            'token' => $token,
        ], 200);
    }
```

```php
    public function login (Request $request) 
    {
       $request->validate([
            'email'      => ['required', 'string', 'email'],  
            'password'   => ['required', 'string'],
        ]);

        $user = User::where('email', $request->email)->first();

        if ($user) {
            if (Hash::check($request->password, $user->password)) {
                $token = $user->createToken('Mega Panel App')->accessToken;

                return response()->json([
                    'user' => new UserResource($user),
                    'token' => $token,
                ], 200);   
            }
        }
        
        return response()->json([
            'message' => 'email or password is incorrect.',
            'errors' => [
                'email' => ['email or password is incorrect.']
            ]
        ], 422);
    }
```

[🔝 Back to contents](#contents)

### **store-service**

app\Http\Controllers\Dashboard\ServiceController.php:

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

app\helpers.php:

```php
function upload_file($request_file, $prefix, $folder_name)
{
    $extension = $request_file->getClientOriginalExtension();
    $file_to_store = $prefix . '_' . time() . rand(10000, 99999) . '.' . $extension;
    $request_file->storeAs('public/' . $folder_name, $file_to_store);
    return $folder_name . '/' . $file_to_store;
}
```
the upload_file() function takes a file, generates a unique filename, stores the file in the specified folder, and returns the relative path of the uploaded file,
it placed into app helper class  provide common functions for various tasks,  These helper functions are globally accessible throughout the application without needing to import or instantiate any specific class.

The function takes three parameters:
1-$request_file: The file obtained from the request.
2-$prefix: A prefix string used to create a unique filename.
3-$folder_name: The name of the folder where the file will be stored.

the function returns the relative path of the uploaded file by concatenating the $folder_name and the unique filename. This path can be used to reference and retrieve the file later.

[🔝 Back to contents](#contents)

### **products-api**

routes\api.php:

These two routes define API endpoints related to product management.
- The first route handles retrieving a collection of all products.
- The second route is used to retrieve a specific product by its id.
The assigned names ('products.all' and 'products.show') I used to reference these routes in my application.

```php
Route::get('products', [ProductController::class, 'index'])->name('products.all');
Route::get('products/{product}', [ProductController::class, 'show'])->name('products.show');
```

app\Http\Controllers\Api\ProductController.php:

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
- index() method:
    This method retrieves all products from the database, including their associated categories.
    The retrieved products are then transformed into a collection of [ProductResource](#product-resource) instances using ProductResource::collection($products).

- show() method:
    This method retrieves a single product from the database, by expecting a Product model instance to be passed as a parameter.
    The method returns the specified product as a JSON response by creating a [ProductResource](#product-resource) instance for the given $product using new ProductResource($product).

### **#product-resource**

app\Http\Resources\ProductResource.php:

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

### **Ckeditor**

The CKEditor library is used to enhance the text editing capabilities of the contact form's description field. CKEditor is a popular WYSIWYG (What You See Is What You Get) text editor that allows users to format and style their text easily.

(/images/Ckeditor.png)

resources\views\dashboard\publication_create.blade.php:

```html
<x-layouts.dashboard>
.
.
.
    <x-slot name="script">
        <script src="https://cdn.ckeditor.com/4.20.2/standard/ckeditor.js"></script>
    </x-slot>
.
.
.

    <form method="post" action="{{ route('publications.store') }}" autocomplete="off" class="form-horizontal"
        enctype="multipart/form-data">
        @csrf
.
.
.
        <div class="row">
            <label class="col-sm-3 col-form-label" for="input-description-en">{{ __('Description [en]') }}</label>
            <div class="col-sm-8">
                <div class="form-group{{ $errors->has('description[en]') ? ' has-danger' : '' }}">
                    <textarea class="ckeditor form-control{{ $errors->has('description[en]') ? ' is-invalid' : '' }}"
                        name="description[en]" id="input-description-en" placeholder="{{ __('Enter Description En') }}"
                        aria-required="true">{!! old('description.en') !!}</textarea>
                    @if ($errors->has('description[en]'))
                        <span id="description-en-error" class="error text-danger"
                            for="input-description-en">{{ $errors->first('description[en]') }}</span>
                    @endif
                </div>
            </div>
        </div>
.
.
.
        <button type="submit" class="btn btn-success">{{ __('Save') }}</button>
    </form>
</x-layouts.dashboard>
<script>
    var uploadUrl = "{{ route('upload') }}";
    var csrfToken = "{{ csrf_token() }}";

    CKEDITOR.replace('input-description-en', {
        filebrowserUploadUrl: uploadUrl + "?_token=" + csrfToken,
        filebrowserUploadMethod: 'form'

    });
</script>
<script>
    var uploadUrl = "{{ route('upload') }}";
    var csrfToken = "{{ csrf_token() }}";

    CKEDITOR.replace('input-description-ar', {
        filebrowserUploadUrl: uploadUrl + "?_token=" + csrfToken,
        filebrowserUploadMethod: 'form'

    });
</script>
```
[🔝 Back to contents](#contents)

### **loadImage**

(/images/loadImage.png)

resources\views\dashboard\slider_create.blade.php:

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