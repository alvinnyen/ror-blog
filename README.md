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