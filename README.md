## Advanced capybara testing.

Let's do some more testing! Hurray!

Once again we'll be doing high-level integration/feature tests and dropping down to the unit/model level when needed. This will be shorter. We'll work on:

1. New Vendor forms
2. New Suya forms
3. Vendor Show page.
4. Vendor updating
5. Vendor and Suya deleting.

#### Let's begin.

1. git clone this repo since I made a few quick styling changes to the html/css and integrated bootstrap.

2. Let's write a high-level test for a new vendor form.

    Again, we haven't written any new forms yet, or a buttons for the new form, or any messages or any html. We're going to be writing a test first, which is sort of like our dream scenario of what we want our app to act like/be like. Our test will drive our development.

    Let's create another test file and separate this new vendor spec from our vendor index specs.

    ```Bash
    rails g rspec:feature new_vendor
    ```

    Inside the spec/features/new_vendors_spec.rb file, lets' write our high level test. Let's think about what we want to happen. Our story:

    1. As a visitor, when I visit the vendors_path and click on a "Register New Vendor"
    2. I can be redirected to a new vendors form that has a "New Vendor Form" heading, and an input box for a name, and a submit button.

    ```rubyonrails
    scenario "a user can visit the vendors index page and click on a button to get to the new vendors form" do
      visit vendors_path
      click_button "Register New Vendor"

      expect(current_path).to eql(new_vendor_path)
      expect(page).to have_selector("h1.new_vendor_header", text: "New Vendor Form")
      expect(page).to have_field("name", value: "name")
      expect(page).to have_button("Submit")
    end
    ```

    Why am I using "scenario" instead of it:  
    [scenario vs it] (https://www.relishapp.com/rspec/rspec-rails/docs/feature-specs/feature-spec)

    >The feature and scenario DSL correspond to describe and it, respectively.
    These methods are simply aliases that allow feature specs to read more as
    customer and acceptance tests.

    Run our tests. Our error:

    ```Bash
    Failure/Error: click_button "Register New Vendor"
     Capybara::ElementNotFound:
       Unable to find button "Register New Vendor"
    ```

    So let's change the top of our index.html.erb to:

    ```html
    <h1> Vendors And Their Suyas</h1>

    <%= link_to "Register New Vendor", new_vendor_path %>

    <div class="vendors">
      <% @vendors.each do |vendor| %>
        <div class="vendor col-sm-4 col-sm-offset-4">
          <p>Vendor name: <%= vendor.name %></p>
          <% vendor.suyas.each do |suya| %>
            <li class="suyas"> I sell <%= suya.meat %> which costs <%= suya.price %>. Spicy: <%= suya.spicy %> %></li>
          <% end %>
        </div>
      <% end %>
    </div>
    ```

    Rerun tests. Our error:

    ```Bash
    Failure/Error: visit vendors_path
     ActionView::Template::Error:
       undefined local variable or method `new_vendors_path' for #<#<Class:0x007fc94d15da88>:0x007fc951a77ae8>
    ```

    Let's delete our old route and replace it with:

    ```rails
    Rails.application.routes.draw do
      resources :vendors
    end
    ```

    Run rake routes:

    ```Bash
        Prefix Verb   URI Pattern                 Controller#Action
        vendors GET    /vendors(.:format)          vendors#index
                POST   /vendors(.:format)          vendors#create
     new_vendor GET    /vendors/new(.:format)      vendors#new
    edit_vendor GET    /vendors/:id/edit(.:format) vendors#edit
         vendor GET    /vendors/:id(.:format)      vendors#show
                PATCH  /vendors/:id(.:format)      vendors#update
                PUT    /vendors/:id(.:format)      vendors#update
                DELETE /vendors/:id(.:format)      vendors#destroy
    ```

    In the above routes for example, the new_vendor_path is a GET request that take you to the /vendors/new url and hits the new action in the vendors controller (or the new instance method in the vendors controller).

    Rerun tests. Our error:

    ```Bash
    Failure/Error: click_button "Register New Vendor"
     Capybara::ElementNotFound:
       Unable to find button "Register New Vendor"
    ```

    I accidentally made the test refer to a link instead of a button. It should be a link since link_to make GET requests by default and button_to makes a POST request by default.

    [Difference between link_to and button_to](http://stackoverflow.com/questions/12475299/ruby-on-rails-button-to-link-to)

    Let's change our test to a click_link:

    ```rails
    scenario "a user can visit the vendors index page and click on a button to get to the new vendors form" do
      visit vendors_path
      click_link "Register New Vendor"

      expect(current_path).to eql(new_vendor_path)
      expect(page).to have_selector("h1.new_vendor_header", text: "New Vendor Form")
      expect(page).to have_field("name", value: "name")
      expect(page).to have_button("Submit")
    end
    ```

    Rerun tests. Our error:

    ```Bash
    Failure/Error: click_link "Register New Vendor"
     AbstractController::ActionNotFound:
       The action 'new' could not be found for VendorsController
    ```

    True dat. Let's fix this. In our VendorsController:

    ```rails
    def new

    end
    ```

    Rerun tests. Our new error...get it?:

    ```Bash
    Failure/Error: click_link "Register New Vendor"
     ActionView::MissingTemplate:
       Missing template vendors/new...
    ```

    In short, we need a vendors/new template.

    Create a new.html.erb template inside app/views/vendors.

    Rerun tests. Our new error:

    ```Bash
    Failure/Error: expect(page).to have_selector("h1.new_vendor_header", text: "New Vendor Form")
       expected to find css "h1.new_vendor_header" with text "New Vendor Form" but there were no matches
    ```
