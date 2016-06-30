### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/22ab5a326159a638c11ff9132c4be966ff654022) Specify the feature for creating a blog post

Red: No route matches [GET] "/blog_posts/new"

We set up the entire acceptance test at once. This test will guide us through the rest of the unit testing and implementation of the feature.

#### spec/features/creating_a_blog_post_spec.rb

```diff
@@ -0,0 +1,22 @@
+# spec/features/creating_a_blog_post_spec.rb
+require 'rails_helper'
+
+describe 'Creating a blog post' do
+
+  it 'saves and displays the resulting blog post' do
+    visit '/blog_posts/new'
+
+    fill_in 'Title', with: 'Hello, World!'
+    fill_in 'Body', with: 'Hello, I say!'
+
+    click_on 'Create Blog Post'
+
+    blog_post = BlogPost.order("id").last
+    expect(blog_post.title).to eq('Hello, World!')
+    expect(blog_post.body).to eq('Hello, I say!')
+
+    expect(page).to have_content('Hello, World!')
+    expect(page).to have_content('Hello, I say!')
+  end
+
+end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/9b417136c921c120a88ea6f2898009678b054251) Add blog posts resource route

Red: uninitialized constant BlogPostsController

We only change enough code to get to the next error message. Getting past the "no route" error only requires creating the route in the routes file.

Rails allows you to "unit test" routes, but for trivial configuration like this, it's fine to let the acceptance test cover it without stepping down to the unit level.

#### config/routes.rb

```diff
@@ -1,3 +1,4 @@
 # config/routes.rb
 Rails.application.routes.draw do
+  resources :blog_posts
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/5e0200a8fe01ba9a568b639f19b6eea3ab0d31a0) Add empty blog posts controller

Red: The action 'new' could not be found for BlogPostsController

We didn't technically need to subclass ApplicationController to get the test to pass; all we needed was a BlogPostsController class. But since we're sure that BlogPostsController will extend ApplicationController, we can safely go ahead and add that code as well.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -0,0 +1,3 @@
+# app/controllers/blog_posts_controller.rb
+class BlogPostsController < ApplicationController
+end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/5b7ca108387fe1cee435a7c09ea8da7b7cda85cb) Add `new` action to blog posts controller

Red: Missing template blog_posts/new

Again, we only add enough code to get the test to pass.

Even though we didn't ask to render a template, Rails' default behavior for a controller action is to render a corresponding template, so that's the error it's running across next.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -1,3 +1,5 @@
 # app/controllers/blog_posts_controller.rb
 class BlogPostsController < ApplicationController
+  def new
+  end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/eb329b9cf22f4eb9145929ccfd9826c42a562912) Add new blog post template

Red: Unable to find field "Title"

The acceptance test is finally able to successfully `visit '/blog_posts/new'` and move on to attempt the next step, which is `fill_in 'Title', with: 'Hello, World!'`.

#### app/views/blog_posts/new.html.erb

```diff
@@ -0,0 +1 @@
+<%# app/views/blog_posts/new.html.erb %>
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/696bf4aaa62790b117df931197334a8f00d5186d) Add fields and submit button to form template

Red: First argument in form cannot contain nil or be empty

<%= form_for @blog_post do |f| %>

In other words, @blog_post is nil but needs to be a model.

This is another case where all we needed to add to get the test to pass was the title field. But this is a reasonable case where you know what the form will consist of, so you can go ahead and create it all. Plus, your tests will never drive out the full markup of your templates, so you'll need to do that by hand anyway.

#### app/views/blog_posts/new.html.erb

```diff
@@ -1 +1,12 @@
 <%# app/views/blog_posts/new.html.erb %>
+<%= form_for @blog_post do |f| %>
+  <div>
+    <%= f.label :title %>
+    <%= f.text_field :title %>
+  </div>
+  <div>
+    <%= f.label :body %>
+    <%= f.text_area :body %>
+  </div>
+  <%= f.submit 'Create Blog Post' %>
+<% end %>
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/3a63ed0cb5b12aa9f9bccaa0ca3c979a9482614c) Specify controller should assign a blog post variable

Inner red: uninitialized constant BlogPost

We're mocking the `new` method on the actual BlogPost constant, so we need that constant to exist.

#### spec/controllers/blog_posts_controller_spec.rb

```diff
@@ -0,0 +1,15 @@
+# spec/controllers/blog_posts_controller_spec.rb
+require 'rails_helper'
+
+describe BlogPostsController do
+
+  describe '#new' do
+    it 'returns a blog post' do
+      blog_post = instance_double(BlogPost)
+      expect(BlogPost).to receive(:new).and_return(blog_post)
+      get :new
+      expect(assigns[:blog_post]).to eq(blog_post)
+    end
+  end
+
+end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/f253bcec6b92abbd51b4635f139c7e2e0a2e854a) Create blog post model and table

Red: expected: #<InstanceDouble(BlogPost) (anonymous)>
          got: nil

expect(assigns[:blog_post]).to eq(blog_post)

In other words, a @blog_post instance variable is not available in the view template, because it wasn't assigned in the controller action.

The reason we don't create a model spec is because it's often not recommended to test-drive trivial features of your model, such as fields and validations. Your acceptance specs should cover these, and there's no need to use unit tests to drive their design, because there's little to no design involved. Model specs are useful for custom validators and other custom methods on the model, including calculated fields and business logic methods.

#### app/models/blog_post.rb

```diff
@@ -0,0 +1,3 @@
+# app/models/blog_post.rb
+class BlogPost < ActiveRecord::Base
+end
```


#### db/migrate/20160223100510_create_blog_posts.rb

```diff
@@ -0,0 +1,9 @@
+# db/migrate/20160223100510_create_blog_posts.rb
+class CreateBlogPosts < ActiveRecord::Migration
+  def change
+    create_table :blog_posts do |t|
+      t.string :title
+      t.text :body
+    end
+  end
+end
```


#### db/schema.rb

```diff
@@ -11,9 +11,14 @@
 #
 # It's strongly recommended that you check this file into your version control system.

-ActiveRecord::Schema.define(version: 0) do
+ActiveRecord::Schema.define(version: 20160223100510) do

   # These are extensions that must be enabled in order to support this database
   enable_extension "plpgsql"

+  create_table "blog_posts", force: :cascade do |t|
+    t.string "title"
+    t.text   "body"
+  end
+
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/1e17d89f646a264480e1703970a8e2743e9d309a) Add blog post variable assignment to controller

This makes the controller spec pass, and moves the acceptance spec on to the next error:

Outer Red: The action 'create' could not be found for BlogPostsController

The acceptance spec now gets to the point of clicking on the 'Create Blog Post' button, but the `create` action the form attempts to submit to doesn't exist.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -1,5 +1,6 @@
 # app/controllers/blog_posts_controller.rb
 class BlogPostsController < ApplicationController
   def new
+    @blog_post = BlogPost.new
   end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/a4d88c0acb02a2fe8ddfcaedb3d829a47a52f37d) Add create method to blog post controller

Outer red: Missing template blog_posts/create

#### app/controllers/blog_posts_controller.rb

```diff
@@ -3,4 +3,7 @@ class BlogPostsController < ApplicationController
   def new
     @blog_post = BlogPost.new
   end
+
+  def create
+  end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/137476286b73930c84e51e20bc109260eb00c2e5) Add blog post create template

Outer red: undefined method `title' for nil:NilClass

expect(blog_post.title).to eq('Hello, World!')

In other words, the blog post was not found in the database.

Note that usually a create method would redirect to a show page, but for the sake of simplicity in this example we're just going to have the create method render the show page directly.

#### app/views/blog_posts/create.html.erb

```diff
@@ -0,0 +1 @@
+<%# app/views/blog_posts/create.html.erb %>
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/e0b217c778073f44836f141f0d3b6cd7084bb98e) Specify controller should create blog post record

Inner Red: BlogPost.create
           expected: 1 time with arguments: ({:title=>"My Title", :body=>"My Body"})
           received: 0 times

We specified the `create` method on the `BlogPost` that the controller nees to call to actually create the blog post, but it's not yet called.

#### spec/controllers/blog_posts_controller_spec.rb

```diff
@@ -12,4 +12,16 @@
     end
   end

+  describe '#create' do
+    it 'creates a blog post record' do
+      expect(BlogPost).to receive(:create).with(title: 'My Title',
+                                                body: 'My Body')
+      post :create, {
+        blog_post: {
+          title: 'My Title',
+          body: 'My Body',
+        }
+      }
+    end
+  end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/d3669d95aa8f0f919cc8fc386cffcb42181c0234) Create blog post record directly from params

This makes the controller spec pass, but the acceptance spec has a related error:

Outer red: ActiveModel::ForbiddenAttributesError

Because the controller spec passes a plain `Hash` in, it passes, but when the real parameters object is passed in from the Rails framework, Rails' security prevents it from being used to set fields on the model unless strong parameters are configured to allow fields through.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -5,5 +5,6 @@ def new
   end

   def create
+    BlogPost.create(params[:blog_post])
   end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/aec1e0b46a3e023e7bd3950065ebe4b8cdc29400) Switch to strong params for blog post creation

Outer red: expected to find text "Hello, World!" in ""

expect(page).to have_content('Hello, World!')

This adds strong params whitelisting to the title and body attributes, which allows the model to be saved and the acceptance spec to proceed past the model checks. Next it checks the output on the page, which is not yet present.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -5,6 +5,12 @@ def new
   end

   def create
-    BlogPost.create(params[:blog_post])
+    BlogPost.create(blog_post_params)
+  end
+
+  private
+
+  def blog_post_params
+    params.require(:blog_post).permit(:title, :body)
   end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/314b9ee198cd2c6cbbb673ad4ab2a580b327d793) Render blog post on create page

Outer red: undefined method `title' for nil:NilClass

<h1><%= @blog_post.title %></h1>

In other words, the @blog_post variable is not available in the view template because it was not assigned in the controller action.

#### app/views/blog_posts/create.html.erb

```diff
@@ -1 +1,6 @@
 <%# app/views/blog_posts/create.html.erb %>
+<h1><%= @blog_post.title %></h1>
+
+<div>
+  <%= @blog_post.body %>
+</div>
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/75cfb6e014996c4871de8273128de15d454f58d1) Specify controller create action should set blog post variable

Inner red: expected: #<InstanceDouble(BlogPost) (anonymous)>
                got: nil

expect(assigns[:blog_post]).to eq(blog_post)

We reproduce the acceptance spec error in the controller spec, so that we can now implement it.

#### spec/controllers/blog_posts_controller_spec.rb

```diff
@@ -23,5 +23,17 @@
         }
       }
     end
+
+    it 'returns the new blog post to the view' do
+      blog_post = instance_double(BlogPost)
+      allow(BlogPost).to receive(:create).and_return(blog_post)
+      post :create, {
+        blog_post: {
+          title: 'My Title',
+          body: 'My Body',
+        }
+      }
+      expect(assigns[:blog_post]).to eq(blog_post)
+    end
   end
 end
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/d10ddecf215cdf3fac5751cb81895b36ce3c1d0f) Add blog post variable assignment to controller create method

At this point, the controller spec passes, and the acceptance spec passes as well. We've successfully allowed the acceptance spec to drive us through implementing our feature, including any necessary lower-level specs.

#### app/controllers/blog_posts_controller.rb

```diff
@@ -5,7 +5,7 @@ def new
   end

   def create
-    BlogPost.create(blog_post_params)
+    @blog_post = BlogPost.create(blog_post_params)
   end

   private
```

### [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/b18d647a2900e0ae2a9aa40020ecb45d65c00f54) Extract duplicate post params to a variable

We identify some duplication in our tests, so we refactor it to extract those duplicate parameters into a variable, to improve our tests' readability.

#### spec/controllers/blog_posts_controller_spec.rb

```diff
@@ -13,26 +13,25 @@
   end

   describe '#create' do
-    it 'creates a blog post record' do
-      expect(BlogPost).to receive(:create).with(title: 'My Title',
-                                                body: 'My Body')
-      post :create, {
+    let(:post_params) {
+      {
         blog_post: {
           title: 'My Title',
           body: 'My Body',
         }
       }
+    }
+
+    it 'creates a blog post record' do
+      expect(BlogPost).to receive(:create).with(title: 'My Title',
+                                                body: 'My Body')
+      post :create, post_params
     end

     it 'returns the new blog post to the view' do
       blog_post = instance_double(BlogPost)
       allow(BlogPost).to receive(:create).and_return(blog_post)
-      post :create, {
-        blog_post: {
-          title: 'My Title',
-          body: 'My Body',
-        }
-      }
+      post :create, post_params
       expect(assigns[:blog_post]).to eq(blog_post)
     end
   end
```