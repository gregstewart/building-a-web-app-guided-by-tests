### writing our first functional test ###
Thanks to Yeoman we don't need to write our very own functional test this is backed in by the initial installation. If you type `grunt test`, you should see the following output:

	Running "test" task

	Running "clean:server" (clean) task
	Cleaning .tmp...OK

	Running "concurrent:test" (concurrent) task
    
    	Running "copy:styles" (copy) task
    
    
    	Done, without errors.
    
    
    	Execution Time (2014-06-14 13:25:27 UTC)
    	loading tasks  4ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 33%
    	copy:styles    7ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 58%
    	Total 12ms
    
	Running "autoprefixer:dist" (autoprefixer) task

	Running "connect:test" (connect) task
	Started connect web server on http://localhost:9001

	Running "mocha:all" (mocha) task
	Testing: http://localhost:9001/index.html

	  ․
	
	  1 passing (2ms)

	>> 1 passed! (0.00s)

	Done, without errors.


	Execution Time (2014-06-14 13:25:26 UTC)
	concurrent:test  1.3s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 46%
	mocha:all        1.5s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 52%
	Total 2.9s

Let's review the tests. 
