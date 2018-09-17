# RoR Blog
應用rails搭建Blog的練習專案，旨在練習Routes及CRUD等相關操作，計劃在未來一週內繼續加上styling和更複雜的資料庫操作，並部署至Heroku讓整個專案更為完整。

## to start the server on cloud9
- `rails server -b $ip[ -p $port]`

## 1. create articles table
- naming convention
    - snake-case for file names
    - camelCase for Model and Controller Name
- step 1.1. generate a migration for creating articles table
    - `rails generate migration ...` # migration operation matters
        - `create_$tables`  
        - ...
        - etc.
        - 在migration file中至少會有change action/method
    - `rails generate migration create_articles`
- step 1.2. run the migration file to truly create the table
    - `rails db:migrate`
        - verify the "table schema" has been generated in $app/db/schema.rb
- others
    - want to update schema after the migration file has been executed / when you make a mistake
        - option 1. `rials db:rollback` for backing to empty schema
        - option 2. (recommanded) create a new migration file when make a mistake, i.e. `rails generate migration add_description_to_articles`
            - use `add_column(tableName, columnName, valueType)` function
                - i.e. `add_column :articles, :description, :text` 
    - 記得在table加上timestamp columnsssss，且欄位名稱必須和規定相同，rails app才能夠正確的追蹤
        - i.e.
            - `add_column :articles, :created_at, :datetime`
            - `add_column :articles, :updated_at, :datetime`
    - migration file 只會被run 1 次，執行過就不會再重複執行

## 2. create the model ($projectName/app/models/article.rb) to communicate with the tables
- remember to follow the naming convention
- 在model建立的當下，rails會 "自動生成" 該model所對應的table的欄位的setters/getters
    - can test it through `rails console`
        - test the connection: `Article.all`
        - `Article` (this means the Article Model) to check the schema of the table (articles)
        - i.e. 
            - `article = Article.new(...)`
                - 或可用資料欄位setter/getter的方式來替代操作, i.e. `article.description = '....'`
                - 或`Article.create(...)` # $Model.create(...) 會直接hit the database，不用另外操作save
            - `article.save` # 最後需要做save所以有的操作才會真正的去hit the database
                - begin the transaction
                - generate the sql query
                - commit the transaction
            - `Article.all` to check the data row has been inserted

## 3. edit and delete a data row
- into `rails console`
    - edit
        - through setter/getter
            - `article = Article.find($id)` # $Model.find($id)
            - `article.description = '....'`
    - delete
        - `article.destroy` # it will return the data row/ model instance that was been deleted
        - `Article.all` to check/verify that the data have been deleted

## 4. about the validation
- maintain data integrity by adding the validation to the model
    - to avoid the article like this: (title: nil, description: nil....)
- 在model中加上欄位的constraints，使能在model instance save時去做檢查，避免nil value存入
    - >> begin transaction
    - >> rollback transaction
        - to find out that why rollback
            - `article.errors.any?` # true
            - `article.errors.full_messages` # ['title can't be blank]
    - !! remember to restart the console by `reload!`

## 5. demonstrates the functionalities with UI
- it needs routes、controllers and views
- lifecycle
    - ![](https://i.imgur.com/khZpa1r.png)
        - @ means it's a instance variable # ???
        - view -> controller -> model
- add the route of `$rootUrl/articles/new`
    - add `resources :articles` to $project/config/routes.rb `# resources :$table`
        - This will add the following routes:
            - routes path HTTP verb link controller#action
            - articles index articles GET /articles articles#index
            - new article new_article GET /articles/new articles#new
            - create article POST /articles articles#create
            - edit article edit_article GET /articles/:id articles#edit
            - update article PATCH /articles/:id articles#update
            - show article article GET /articles/:id articles#show
            - delete article DELETE /articles/:id articles#destroy
        - it will auto-generate the CRUD routes for you
            - type `rails routes` to check the existence routes
                - ![](https://i.imgur.com/1uuKIia.png)
- refresh the page and proceed after check the error (uninitialized constant ArticlesController)
- add the controller (articles_controller.rb/ArticlesController)
    - it need the "new action" # based on the url/route
        ```
            # $projectName/app/controllers/articles_controller.rb
        
            class ArticlesController < ApplicationController
                def new
                end
            end
        ```
- refresh the page and proceed after check the error (ArticlesController#new is missing a template for this request format and variant. ....)
- add the templates in $porject/app/views/articles folder
    - 因為是articles_controller，所以會去views/articles底下找templates/views by request
    - add $project/app/views/articles/new.html.erb `# $actionName.html.erb`
        ```
            # app/views/articles/new.html.erb
        
            <h1> create an article </h1>
        ```
    - 增加表單
        - it's model based form, rails provides some useful tools that you can use with form_helpers (check http://guides.rubyonrails.org/form_helpers.html)
            ```
                # app/views/articles/new.html.erb
            
                <h1> create an article </h1>
                
                <%= form_for @article do |f| %>
                
                <% end %>
            ```
- refresh the page and proceed after check the error (First argument in form cannot contain nil or be empty)
    - 不認得template中的@article => 要在controller中去初始化 # why in controller? => 思考整個render flow 
        ```
            # app/controllers/articles_controller.rb
            
            class ArticlesController < ApplicationController
                def new
                    @article = Article.new
                end
            end
        ```
    - complete the form
        ```
           # app/views/articles/new.html.erb
            
            <h1> create an article </h1>
                
            <%= form_for @article do |f| %>
                <p>
                    <%= f.label :title %><br/>
                    <%= f.text_field :title %>
                </p>
                
                <p>
                    <%= f.label :description %><br/>
                    <%= f.text_area :description %>
                </p>
                
                <p>
                    <%= f.submit %> 
                </p>
                
            <% end %>
        ```
- refresh the page then click the button and proceed after check the error (The action 'create' could not be found for ArticlesController.)
- add the create action to articles_controller
    ```
        # app/controllers/articles_controller.rb    
        
        ...
        
            def create
                
            end
    
    ```
- refresh the page then click the button and proceed after check the error
    - ![](https://i.imgur.com/DGRluxp.png)
- just render what you get in the controller
    ```
        # app/controllers/articles_controller.rb    
        
        ...
        
            def create
                render plain: params[:article].inspect
            end
    
    ```
- refresh and test it
- save the value by the controller and the action article#create
    ```
        # app/controllers/articles_controller.rb    
        
        ...
        
            def create
                # render plain: params[:article].inspect
                
                @article = Article.new(article_params)
                @article.save
            end
            
            private
                def article_params
                    params.require(:article).permit(:title, :description)
                end
    
    ```
- refresh the page then click the button, 不用管error，只要在rails console確認資料真的有存database就好
    - enter the values in the browser  
    - `Article.all` 
    - enter the values in the browser
    - `reload!`
    - `Article.all` 
- redirect to articles_show(@article)
    - type `rails routes` to check the route articles#show
        ```
            # app/controllers/articles_controller.rb    
        
            ...
        
            def create
                # render plain: params[:article].inspect
                
                ....
                redirect_to article_show(@article)
            end
    
        ```
    - the article_show will be introduced in the next chapter
    
        
   
