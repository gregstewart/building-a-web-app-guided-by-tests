With that all done let's go ahead and install [Yeoman](http://yeoman.io/) as it gives us a quick way to get started with our new app. Type the following: `npm install -g yo`

Next we'll install the webapp generator for Yeoman: `npm install -g generator-webapp`. Once everything is installed let's scaffold our app: `yo webapp` ([see appendices for the full output](Yeoman output) - please note that I opted to inlcude SASS, Modernizr and Bootstrap). 

To verify that everything has installed properly go ahead and type `grunt`, if everything is happy you should see a bunch of output that ends with something like this:

	Execution Time (2014-06-14 13:15:27 UTC)
	jshint:all          98ms  ▇▇ 1%
	concurrent:test     1.3s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 14%
	mocha:all           1.5s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 16%
	concurrent:dist     2.7s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 28%
	autoprefixer:dist  300ms  ▇▇▇▇ 3%
	cssmin:generated   108ms  ▇▇ 1%
	uglify:generated      2s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 21%
	modernizr:dist      1.2s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 13%
	Total 9.4s

Typing grunt `grunt serve` should start our server at [http://localhost:9000](http://localhost:9000) and open up the browser.

![App running using the local server](Screenshot 2014-06-14 14.18.44.png)

We are nearly done, let's commit this to our local repository. Typing `git status` should show you all changes currently pending:

	# On branch master
	#
	# Initial commit
	#
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	.editorconfig
	#	.gitattributes
	#	.gitignore
	#	.jshintrc
	#	Gruntfile.js
	#	app/
	#	bower.json
	#	package.json
	#	test/
	nothing added to commit but untracked files present (use "git add" to track)

Let's stage all of the untracked files: `git add .` and now let's commit these changes `git commit -m "Initial commit, Yeoman base set up"`
