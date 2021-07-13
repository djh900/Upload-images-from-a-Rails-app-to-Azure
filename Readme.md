# Image Upload with Microsoft Azure

Microsoft Azure is a cloud computing service which competes with AWS. In this readme, I will explain how to upload images from Rails apps to Microsoft Azure instead of AWS.

### Creating a Storage Account

Firstly, you will need to sign up for Azure [here](https://azure.microsoft.com/en-au/free/search/?&ef_id=EAIaIQobChMIo5iNiNbe8QIVGJhmAh2L3wbrEAAYASAAEgJZ0_D_BwE:G:s&OCID=AID2200144_SEM_EAIaIQobChMIo5iNiNbe8QIVGJhmAh2L3wbrEAAYASAAEgJZ0_D_BwE:G:s&gclid=EAIaIQobChMIo5iNiNbe8QIVGJhmAh2L3wbrEAAYASAAEgJZ0_D_BwE). You can sign up using your GitHub or Microsoft account.

Once you're signed in, you will be able to access the Azure console:

![Azure Console](/images/image1.png)

If this is your first time signing in to Azure, you will be taken to the Quick Start page. Select 'Home' on the top left corner to go to the console:

![Quick Start Page](/images/image3.png)

From here, select Storage Accounts:

![Storage Accounts in Azure Console](/images/image2.png)

If you can't find this link on the console, search for 'Storage Accounts' in the searchbar at the top of the page.

From the storage accounts page, we're going to create a new storage account:

![Storage Accounts Overview Page](/images/image4.png)

On the create storage account page that will come up, there are a few options.

Firstly, for the Subscription, select 'Free Trial'. Azure comes with a 1-year free trial, and selecting this will also ensure you aren't charged, even if you exceed the allocated data cap.

Next, you will need to create a new Resource Group. This is a container that holds multiple Azure resources together (say, for example, you could have a virtual machine and cloud storage account together in the one resource group).

![Create a storage account 1](/images/image5.png)

Next, you will need to input the details of the new storage account. Under 'Instance details', give the storage account a name (lowercase letters and numbers only). This name needs to be unique. You can leave the settings for Region, Performance and Redundancy to their default values.

![Create a storage account 2](/images/image6.png)

You can add other settings to the new storage account, however for the purpose it is being used here we won't need to. Select 'Review and Create' at the bottom left of the screen. This will take you to the review page. From there, you can select 'Create' to create the new storage account.

![Storage account create review](/images/image7.png)

Creating the storage account will take approximately 60 seconds to complete. If everything went well, you will receive a 'Deployment Complete' message and you can visit the new storage account with the 'Go to resource' button.

![Deployment in progress](/images/image8.png)

![Deployment complete](/images/image9.png)

![Storage account overview](/images/image10.png)

### Creating a container

Now, create a container to store the images from your Rails app in:

![Create container 1](/images/image11.png)

![Create container 2](/images/image12.png)

Give the container a name and ensure the access level is set to private, then click create:

![Create container 3](/images/image13.png)

The created container will now be shown in the storage account containers:

![Create container 4](/images/image14.png)

You're now ready to start uploading images from a Rails App to Azure.

### Implementing image upload in a Rails app

If you have a Rails app setup and ready to go, you can skip this step. Otherwise, let's create a simple Rails app with a scaffold using the following terminal commands. This creates a basic app with CRUD actions to store books with titles, as well as setting up active storage so we can start uploading images.

```
$ rails new azure_test_app
$ cd azure_test_app
$ rails db:create
$ rails g scaffold books title:string
$ rails active_storage:install
$ rails db:migrate
```

Open app with your code editor.

Let's create a new form field to allow images to be uploaded.

In the app/views/books/\_form.html.erb file, above the submit button, add the following:

```
<div class="field">
  <%= form.label :image %>
  <%= form.file_field :image, accept: 'image/png, image/jpg, image/jpeg' %>
</div>
```

Your \_form.html.erb file should now look like this:

![_form.html.erb file](/images/image15.png)

We now need to permit the image through the strong params filter in the books controller.

In the app/controllers/books_controller.rb file, go to the book_params function down the bottom. Add :image to the permit field:

![books_controller.rb file](/images/image16.png)

Next, we need to give the book model the attribute image.

In the app/models/book.rb file, add the line `has_one_attached :image` to the Book class:

![book.rb model file](/images/image17.png)

We can now upload images to the Rails app!

Finally, let's display these images on each book's show page.

In the app/books/views/show.html.erb file, add some embedded ruby to show each book's uploaded image:

`<%= image_tag @book.image if @book.image.attached? %>`

![show.html.erb file](/images/image18.png)

###Uploading images to Azure

You should now have an Azure container setup, as well as a Rails app with an image upload feature.

The final step is to set up the Rails app so its images are uploaded to the Azure container rather than the its local storage.

Firstly, go to the config/storage.yml file and uncomment the section for Microsoft Azure:

![storage.yml file 1](/images/image19.png)

We will need to change the storage_account_name and container values so the app knows where to upload these images to in Azure.

You want to use the same names for the storage account and container created in Azure. For the example shown above, we used examplerailsaccount1 for the storage account name and railsappimagescontainer for the container, so we will change those values in the storage.yml file accordingly:

![storage.yml file 2](/images/image20.png)

We now need to provide our app credentials to upload images to the Azure container.

These keys are found on the storage account page in Azure, in the menu on the left hand side under 'Security + networking':

![Access keys 1](/images/image21.png)

We want to grab the first access key and add it to our Rails credentials file.

Copy the key found on that page:

![Access keys 2](/images/image22.png)

From the Rails app's root folder in the terminal, open the credentials file with the command

`$ EDITOR='code --wait' rails credentials:edit`

This will open the app's credentials file in VS Code:

![Credentials file 1](/images/image23.png)

Here, we will add the key copied above:

![Credentials file 2](/images/image24.png)

We now need to change the environments files to tell the app to upload images to Azure instead of uploading them locally.

In config/environments/production.rb, change

`config.active_storage.service = :local`

to

`config.active_storage.service = :microsoft`

![config file](/images/image25.png)

You may wish to do the same in config/environments/development.rb to upload images to Azure while you're devloping the app.

Finally, we need to install the azure-storage-blob gem. You can do this in the app's root folder in terminal with

`$ bundle add azure-storage-blob`

That's it! Images uploaded to the app will now be stored in Microsoft Azure.
