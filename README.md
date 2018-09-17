# RoR Blog

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
            - `article = new Article(....)` 
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
