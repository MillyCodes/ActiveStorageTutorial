# ActiveStorageTutorial
Markup file for a presentation on using Active Storage w/ Rails (Saving Files in Rails)


# Active Storage in Rails
## Overview

Active Storage is a new feature of Ruby on Rails 5.2. Active Storage allows Rails developers to manage file uploads, cloud storage, and document management tied to models throughout apps. It attaches pointers to the Active Record object and will upload it asynchronously.
    
:::info
Rails 5.2 and later: Active Storage included
Rails 5.1.6 and older: requires `gem`
:::

## Some Features
### Mirror Service
Allows syncing of files between multiple cloud storage services.

For example, in `config/storage.yml` we have this block:
```ruby
development:
  service: Mirror
  primary: amazon
  mirrors:
    - azure
    - google
```

In this example, the mirror service would first upload files to **Amazon** since it is set as the primary. Then, it would push the same files to **Azure** and **Google Cloud**. File removal takes place in the same order. This can be helpful when switching between cloud services or when creating backups.


### Direct Uploads
Since Active Storage comes with a Javascript library (`activestorage.js`) we can upload files from the front end of our site to cloud storage directly.

Some events used by this library are: 
```bash
direct-upload:start
direct-upload:initialize
direct-upload:progress
```

### Asynchronous Upload
Doesn’t require adding any background job to upload files asynchronously. It uses Active Job to upload files to the cloud.

## Installation

1. Within your Rails app, look in the following folders: 

`config/environments/development` and `config/environments/production`

Ensure the following is included:  `config.active_storage.service = :local`. This tells the app to save your uploads locally.

2. To install into your application run:

```bash
rails active_storage:install
```

This generates an `active_storage_blobs` and an `active_storage_attachments` table.

Run: 
```bash
rails db:migrate
```

Once you configure it you can go to your models, and define your attachments. If you have a model for User and need to upload the profile picture of that user and allow them to upload other files: 

```ruby
class User < ApplicationRecord
    has_one_attached :profile_picture
    has_many_attached :uploads
end
```

The `has_one_attached` method maps a one to one relationship between the Active Record Object and the uploaded file. The `has_many_attached` maps a one to many relationship.

When you have a form for the user using form_with you can simply add fields for the attachments within that form: 
```ruby
<%= form.file_field :profile_picture %>
<%= form.file_field :uploads, multiple: true %>
```

We also have to create an action in the user controller that takes in the user params, here we just added :profile_picture and :uploads into it: 
```ruby
def create
  @user = User.create(user_params)
end

private

def user_params
  params.require(:user).permit(:email, :password, :profile_picture, uploads: [])
end
```

*Notice uploads parameter has many so we use array notation here.

One cool thing about this, is that if you go to the index page that might list the users, all you have to do to display an image is say `<%= image_tag @user.profile_picture %> `


## Demo
create new app
`rails new myapp --database=postgresql`

Scaffold for Demo purposes
`rails g scaffold post title body:text`

---

`rails db:create`
`rails db:migrate`

---

`rails active_storage:install`
`rails db:migrate`

creates:
- active_storage_blobs
- active_storage_attachments

---

In `app/models/post.rb`, update it as follows:

```ruby
class Post < ApplicationRecord
  has_one_attached :file
end
```

`:file` can be called anything you prefer, can be application specific (pdf, presentations, image, etc)

---

in `app/views/posts/_form.html.erb`, add an input for the file.

```ruby
<div class="field">
  <%= form.label :file %>
  <%= form.file_field :file %>
</div>
```
---

Whitelist `:file` in `posts_controller.rb`
```ruby
def post_params
  params.require(:post).permit(:title, :body)
end
```
Add:
```ruby
def post_params
  params.require(:post).permit(:title, :body, :file)
end
```


in the show file add
```ruby
<%= link_to 'file', @post.file %>
<%= link_to 'file', @post.file, download: '' %>
```

## Additional Resources

* [Active Storage Overview](https://guides.rubyonrails.org/active_storage_overview.html)
* [Active Storage README](https://github.com/rails/rails/blob/d3893ec38ec61282c2598b01a298124356d6b35a/activestorage/README.md)
* [DEANOUT on YouTube: Active Storage Tutorial Series](https://www.youtube.com/channel/UCRQv-3VvPT9mArF5RfrlpKQ)
* [Tutorial on how to use Active Storage on Rails 5.2](https://www.engineyard.com/blog/active-storage)

