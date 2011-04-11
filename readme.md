# MacRuby HTTP Wrapper

Download helper making file download trivial.
 
The result of the download can be processed by a block or a delegator.
A block will always take precedence over a delegator, meaning that you can't
pass a delegator and use a block, you have to choose which approach
you want to use.
NOTE: in the case of a non blocking download, the delegation block can be called multiple times
due to a bug in Cocoa. (or maybe it's an undocumented feature ;)).

## Parameters

url<String>:: url of the resource to download
  
options<Hash>:: options used for the query, see +MacRubyHTTP::Query.initialize+ for more details

&block<block>:: block which is called when the file is downloaded

## Options
:method<String, Symbol>:: An optional value which represents the HTTP method to use

:payload<String>::    - data to pass to a POST, PUT, DELETE query.

:delegation<Object>:: - class or instance to call when the file is downloaded.
                        the handle_query_response method will be called on the
                        passed object

:save_to<String>::    - path to save the response to

:credential<Hash>::   - should contains the :user key and :password key
                        By default the credential will be saved for the entire session

## TODO

:progress <Proc>::    Proc to call everytime the downloader makes some progress

:cache_storage

## Examples
    download("http://www.macruby.org/files/MacRuby%200.4.zip", {:save_to => '~/tmp/macruby.zip'.stringByStandardizingPath}) do |macruby|
      NSLog("file downloaded!")
    end

    download "http://macruby.org" do |mr|
      # The response object has 3 accessors: status_code, headers and body
      NSLog("status: #{mr.status_code}, Headers: #{mr.headers.inspect}")
    end
 
    path = File.expand_path('~/macruby_tmp.html')
    window :frame => [100, 100, 500, 500], :title => "HotCocoa" do |win|
      download "http://macruby.org", {:save_to => path} do |homepage|
        win << label(:text => "status code: #{homepage.status_code}", :layout => {:start => false})
        win << label(:text => "Headers: #{homepage.headers.inspect}", :layout => {:start => false})
      end
    end

    download "http://macruby.org/users/matt", :delegation => @downloaded_file, :method => 'PUT', :payload => {:name => 'matt aimonetti'}
   
    download "http://macruby.org/roadmap.xml", :delegation => self

    download "http://localhost:5984/couchrest-test/", :method => 'POST', :payload => '{"user":"mattaimonetti@mail.com","zip":92129}', :delegation => self

    download("http://yoursite.com/login", {:credential => {:user => 'me', :password => 's3krit'}}) do |test|
      NSLog("response received: #{test.headers} #{test.status_code}")
    end
 
    # We can also do the same thing but synchronously and block the runloop  
    download "http://macruby.org", :immediate => true, :save_to => '~/tmp/site.html'.stringByStandardizingPath do |mr|
      p "file downloaded, let's continue"
    end
