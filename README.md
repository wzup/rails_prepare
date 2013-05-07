rails_prepare
=============

PHP's [PDO::prepare](http://php.net/manual/en/pdo.prepare.php)-like method. Prepares an SQL statement to be executed by [exec_*](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/DatabaseStatements.html#method-i-exec_query) methods of Rails' ActiveRecord.

Rails do not have such a useful method as PHP's PDO::prepare. See this [answer](http://stackoverflow.com/a/16125655/1114926), if you are curious.

So, this method allows you to prepare your SQLs like in PHP.

But it is not so powerful yet. It only deals with (?) (question marks) as parameters in sql-statements. No (:name) named parameters yet.

It works with MySQL and PostgreSQL. Do not know about others SQL-based databases.


What is this
------------
You provide:

    sql = SELECT * FROM tab WHERE val_1 = ? AND val_2 = ? LIMIT ?
    params = ["foo", "bar", 5]
    
You get:

    prepare(sql, params) #=> "SELECT * FROM tab WHERE val_1 = 'foo' AND val_2 = 'bar' LIMIT 5"

You have to provide parameners in an array with the same sequence as they appear in your sql-statement.
    
    
How to use
----------
For example, you can place this code in your ````\app\controllers\application_controller.rb````:

    before_filter(:setPrepare)
    
    def setPrepare
    	class << ActiveRecord::Base
    		def _prepare(sql, params)
    			params.each { |i|
    				case true
    				when i.instance_of?(String)
    					i = "'#{i}'"
    				when i.instance_of?(Numeric)
    					i = i.to_s
    				when i.nil?
    					i = "NULL"
    				end
    				sql = sql.sub(/\?/, i.to_s)
    			}
    			return sql
    		end
    	end
    end
    
Now ```_prepare``` is available in all your models.

For example, this is the way you can use ```_prepare``` in your models, eg. in ```\app\models\product.rb```:

    def self.doSmth(var_1, val_2)
      sql = self._prepare("INSERT INTO products (name, price) VALUES (?, ?)", [val_1, val_2])
      self.connection().exec_insert(sql)
    end
    
And this is the way you can call it from within one of your controllers, eg. in ```\app\controllers\products_controller.rb```
    
    def update
       Product.doSmth(params["prod_name"], params["prod_price"])
    end






