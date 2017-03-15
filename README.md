# Accessing Amazon Athena with JRuby

Access Amazon Athena using a Java Database Connectivity (JDBC) driver with JRuby.  For information on other connections or to download the Athena driver 
please refer to the [documentation](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html).

**Things to note**

 * By default, you can run five (5) queries concurrently from an account. You can request a service limit increase to raise this limit.


## Install macOS

Prerequisites: RVM

    rvm install jruby-9
    rvm @global do gem install bundler
    bundle

## database.yml

1. JDBC URL Format
    
    The format of the JDBC connection string for Athena is the following: 
    
    `url: jdbc:awsathena://athena.REGION.amazonaws.com:443`
    
    Current REGION values are us-east-1 and us-west

2. **s3_staging_dir**: The Amazon S3 location to which your query output is written. The JDBC driver then asks Athena to read the results and provide rows of data back to the user.

3. **user** and **password**: Use your AWS ACCESS KEY AND SECRET

## Sample

I'm still looking into how [JRuby handles class paths](https://github.com/jruby/jruby/wiki/ClasspathAndLoadPath) but for this example you can export it.

    export CLASSPATH=lib/

Fire up interactive ruby and run the following commands

    bundle exec irb
    
    # Hack:  This code now lives in the listen gem I guess.  See @RickCarlino comment
    # https://github.com/jruby/activerecord-jdbc-adapter/issues/700
    module ActiveRecord
      module ConnectionAdapters
        class Column
          TRUE_VALUES = [true, 1, '1', 't', 'T', 'true', 'TRUE', 'on', 'ON'].to_set
          FALSE_VALUES = [false, 0, '0', 'f', 'F', 'false', 'FALSE', 'off', 'OFF'].to_set
          module Format
            ISO_DATE = /\A(\d{4})-(\d\d)-(\d\d)\z/
            ISO_DATETIME = /\A(\d{4})-(\d\d)-(\d\d) (\d\d):(\d\d):(\d\d)(\.\d+)?\z/ 
          end
        end
      end
    end
    
    require 'active_record'
    require 'AthenaJDBC41-1.0.0'
    require 'activerecord-jdbc-adapter'

    CONFIG = YAML.load(ERB.new(open('config/database.yml').read).result)["development"].symbolize_keys!
    connection = ActiveRecord::Base.establish_connection CONFIG
    ActiveRecord::Base.connection.execute("SELECT count(*) FROM my_database.my_table")
    # => [{"count"=>2271855067}] 
    
    
