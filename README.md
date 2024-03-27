## Step #1: Install Laravel

Install the latest Laravel version i.e. Laravel 11. To do so go to the project directory and run the command:

`composer create-project --prefer-dist laravel/laravel`

##  Step #2: Create Package Directory

create folder from laravel root directory with this structure:

### /packages/prakash/todolist/src

##  Step #3: Composer Initiation 
Every package should have a “composer.json” file, which will contain all the packages and their dependencies. Using terminal, navigate to our package name folder, in this chapter is **packages/prakash/todolist**, and run following command:

`composer init`

You will be prompted for details about the package. You can skip by pressing enter and it will intake the default values. You can change this information later in the "composer.json" file.

Add below dependencies and version.

laravelcollective/html

	{
		"name": "prakash/todolist",
		"description": "You can create the to-do-list of your task.",
		"authors": [
			{
				"name": "Jhon Duo",
				"email": "john@example.com"
			}
		],
		"minimum-stability": "dev"
	}

## Step #4: Load the Package from the Main Composer.JSON File 
Now, the "composer.json" file for every Laravel application is present in the root directory. We need to make our package visible to the application.

Add the namespace of our package in "autoload-dev > psr-4"

    "autoload-dev": {
        "psr-4": {
            "App\\": "app/",
            "Prakash\\Todolist\\": "packages/prakash/todolist/src/"
        }
    },
Then we have to autoload our package using composer command as following: composer dump-autoload

## Step #5: Create a Service Provider for Package

Simply create a TodolistServiceProvider.php class inside src folder. Don't forget to use namespace based on vendor that we’ve created before.

	<?php

	namespace Prakash\Todolist;

	use Illuminate\Support\ServiceProvider;

	class TodolistServiceProvider extends ServiceProvider
	{
		/**
		 * Bootstrap the application services.
		 *
		 * @return void
		 */
		public function boot()
		{

		}

		/**
		 * Register the application services.
		 *
		 * @return void
		 */
		public function register()
		{

		}
	}
To identify namespace, add below psr-4 in compposer.json inside packages/prakash/todolist

	{
		"name": "prakash/todolist",
		"description": "You can create the to-do-list of your task.",
		"type": "library",
		"license": "MIT",
		"authors": [
			{
				"name": "Prakash Joshi",
				"email": "developer.prakashjoshi@gmail.com"
			}
		],
		"minimum-stability": "dev",
		"require": {},
		"autoload": {
			"psr-4": {
				"Prakash\\Todolist\\": "src/"
			}
		}
	}

Next, we need to add package service provider to config/app.php inside providers array.

Prakash\Todolist\TodolistServiceProvider::class,
## Step #6: Create the Migration create the migration using the following artisan command:Step #6: Create the Migration create the migration using the following artisan command:

`php artisan make:migration create_task_table --create=tasks`

The migration is created in this location "database/migration/". We will move this migration file into our package to "packages/prakash/todolist/src/migrations/".

Now, we can modify this migration file add the columns for our table.

	<?php
	use Illuminate\Support\Facades\Schema;
	use Illuminate\Database\Schema\Blueprint;
	use Illuminate\Database\Migrations\Migration;

	class CreateTaskTable extends Migration
	{
		/**
		 * Run the migrations.
		 *
		 * @return void
		 */
		public function up()
		{
			Schema::create('tasks', function (Blueprint $table) {
				$table->increments('id');
				$table->string('name');
				$table->timestamps();
			});
		}

		/**
		 * Reverse the migrations.
		 *
		 * @return void
		 */
		public function down()
		{
			Schema::dropIfExists('tasks');
		}
	}
## Step #7: Create the Model for the Table

Run the following artisan command: 
`php artisan make:model Task `

Now, move the "Task.php" file from app/Task.php to our package folder packages/prakash/todolist/src/Task.php. And again, don’t forget to change the namespace of the file to "Prakash\Todolist".

	<?php
	namespace Prakash\Todolist;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
		protected $table = 'tasks';

		protected $fillable = [
			'name',
		];
	}
## Step #8: Create a Controller Let’s create the controller by running the artisan command:

`php artisan make:controller TaskController`
Next, move the controller (TaskController) from app/Controllers/TaskController.php to packages/Prakash/todolist/Controllers/TaskController.php and change the namespace to "Prakash\Todolist".

	<?php

	namespace Prakash\Todolist;

	use App\Http\Controllers\Controller;
	use Request;
	use Prakash\Todolist\Task;

	class TodolistController extends Controller
	{
		public function index()
		{
			return redirect()->route('task.create');
		}

		public function create()
		{
			$tasks = Task::all();
			$submit = 'Add';
			return view('prakash.todolist.list', compact('tasks', 'submit'));
		}

		public function store()
		{
			$input = Request::all();
			Task::create($input);
			return redirect()->route('task.create');
		}

		public function edit($id)
		{
			$tasks = Task::all();
			$task = $tasks->find($id);
			$submit = 'Update';
			return view('prakash.todolist.list', compact('tasks', 'task', 'submit'));
		}

		public function update($id)
		{
			$input = Request::all();
			$task = Task::findOrFail($id);
			$task->update($input);
			return redirect()->route('task.create');
		}

		public function destroy($id)
		{
			$task = Task::findOrFail($id);
			$task->delete();
			return redirect()->route('task.create');
		}
	}
## Step #9: Create a Routes File

Create a new file in "prakash/todolist/src" folder and give the name "routes.php". Define the routes we are going to use in our package.

	<?php
	Route::resource('/task', 'Prakash\Todolist\TodolistController');

## Step #10: Create the Views

To create views, we have to create a "views" folder under "prakash/todolist/src/". Now, create a file for each required view under this folder.

We’ll be creating two views:

1. - app.blade – for each to-do
2. - list.blade – for the to-do list.

Content to be added under app.blade.php:

	<!DOCTYPE html>
	<html>
	<head>
		<title>TO DO List</title>
		<link type="text/css" rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

	</head>
	<body>

		<div class="container">
			@yield('content')
		</div>

		<script type="text/javascript" src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
		<script type="text/javascript" src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

	</body>
	</html>
Add the following under list.blade.php:

	@extends('prakash.todolist.app')
	@section('content')
		@if(isset($task))
			<h3>Edit : </h3>
			{!! Form::model($task, ['route' => ['task.update', $task->id], 'method' => 'patch']) !!}
		@else
			<h3>Add New Task : </h3>
			{!! Form::open(['route' => 'task.store']) !!}
		@endif
			<div class="form-inline">
				<div class="form-group">
					{!! Form::text('name',null,['class' => 'form-control']) !!}
				</div>
				<div class="form-group">
					{!! Form::submit($submit, ['class' => 'btn btn-primary form-control']) !!}
				</div>
			</div>
		{!! Form::close() !!}
		<hr>
		<h4>Tasks To Do : </h4>
		<table class="table table-bordered table-striped">
			<thead>
				<tr>
					<th>Name</th>
					<th>Action</th>
				</tr>
			</thead>
			<tbody>
				@foreach($tasks as $task)
					<tr>
						<td>{{ $task->name }}</td>
						<td>
							{!! Form::open(['route' => ['task.destroy', $task->id], 'method' => 'delete']) !!}
								<div class='btn-group'>
									<a href="{!! route('task.edit', [$task->id]) !!}" class='btn btn-default btn-xs'><i class="glyphicon glyphicon-edit"></i></a>
									{!! Form::button('<i class="glyphicon glyphicon-trash"></i>', ['type' => 'submit', 'class' => 'btn btn-danger btn-xs', 'onclick' => "return confirm('Are you sure?')"]) !!}
								</div>
							{!! Form::close() !!}
						</td>
					</tr>
				@endforeach
			</tbody>
		</table>
	@endsection

## Step #11: Update the Service Provider to Load the Package 
We’ve come to the penultimate step – loading the routes, migrations, views, and so on. If you want a user of your package to be able to edit the views, then you can publish the views in the service provider.

	<?php
	namespace Prakash\Todolist;

	use Illuminate\Support\ServiceProvider;

	class TodolistServiceProvider extends ServiceProvider
	{
		/**
		 * Bootstrap the application services.
		 *
		 * @return void
		 */
		public function boot()
		{
			$this->loadRoutesFrom(__DIR__.'/routes.php');
			$this->loadMigrationsFrom(__DIR__.'/migrations');
			$this->loadViewsFrom(__DIR__.'/views', 'todolist');
			$this->publishes([
				__DIR__.'/views' => base_path('resources/views/prakash/todolist'),
			], 'views');
		}

		/**
		 * Register the application services.
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->make('Prakash\todolist\TodolistController');
		}
	}
Now you can publish the views by the artisan command:

`php artisan vendor:publish --tag=views`

The above command will create the folder of your package under the views folder "/resources/views/prakash/todolist/". Now user can change the view of the screen.

## Step #12: Package Discovery 
Add extra section in composer.json file packages/prakash/todolist

	{
		"name": "prakash/todolist",
		"description": "You can create the to-do-list of your task.",
		"type": "library",
		"license": "MIT",
		"authors": [
			{
				"name": "Jhon Duo",
				"email": "john@example.com"
			}
		],
		"minimum-stability": "dev",
		"require": {},
		"autoload": {
			"psr-4": {
				"Prakash\\Todolist\\": "src/"
			}
		},
		"extra": {
			"laravel": {
				"providers": [
					"Prakash\\Todolist\\TodolistServiceProvider"
				]
			}
		}
	}
