# laravel-breadcrumbs-docs

**app/Http/Controllers/CategoryController.php**

```
<?php

namespace App\Http\Controllers;

use App\Models\Category;

class CategoryController extends Controller
{
    public function category($id)
    {
        $categories = Category::where('id', $id)->get();

        foreach($categories as $category) {
            $c = implode(' > ', $category->path);

            var_dump($c);
        }
    }
}
```

**app/Http/Models/Category.php**

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    /*
    |--------------------------------------------------------------------------
    | GLOBAL VARIABLES
    |--------------------------------------------------------------------------
    */

    protected $table = 'categories';
    protected $primaryKey = 'id';
    public $timestamps = true;
    // protected $guarded = ['id'];
    protected $fillable = ['name', 'parent_id'];
    // protected $hidden = [];
    // protected $dates = [];
    //protected $translatable = ['title', 'introtext', 'content', 'extras'];
    //protected $fakeColumns = ['extras'];
    //protected $casts = [];

    // One level child
    public function child()
    {
        return $this->hasMany(Category::class, 'parent_id');
    }

    // Recursive children
    public function children()
    {
        return $this->hasMany(Category::class, 'parent_id')
            ->with('children');
    }

    // One level parent
    public function parent()
    {
        return $this->belongsTo(Category::class, 'parent_id');
    }

    // Recursive parents
    public function parents() {
        return $this->belongsTo(Category::class, 'parent_id')
            ->with('parent');
    }

    public function getPathAttribute()
    {
        $path = [];
        if ($this->parent_id) {
            $parent = $this->parent;
            $parent_path = $parent->path;
            $path = array_merge($path, $parent_path);
        }
        $path[] = '<a href="' . $this->id . '-' . $this->slug . '">' . $this->name . '</a>';
        return $path;
    }

}
```

**routes/web.php**

```
Route::get('/category/{id}', [App\Http\Controllers\CategoryController::class, 'category'])->name('category');
```

**database/migrations/2023_06_30_000000_create_categories_table.php**

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateCategoriesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug');
            $table->integer('parent_id')->default(0);
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
        Schema::dropIfExists('categories');
    }
}
```
