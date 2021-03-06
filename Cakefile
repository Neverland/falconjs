
# Building Falcon requires coffee-script, uglify-js, wrench, and sass. For
# help installing, try:
#
# `npm install coffee-script uglify-js wrench`
# `gem install sass`
#
# Original Cake file from Chosen.js - modified for our use
#   https://github.com/harvesthq/chosen/blob/master/Cakefile
fs              	= require 'fs'
{spawn, exec}   	= require 'child_process'
CoffeeScript    	= require 'coffee-script'
UglifyJS 			= require("uglify-js")
wrench          	= require 'wrench'

# Get the version number
version_file = 'VERSION'
version = if fs.existsSync( version_file ) then "#{fs.readFileSync(version_file)}".replace( /[^0-9a-zA-Z.]*/gm, '' ) else ""
version_tag = -> "v#{version}"
build_file = "build.json"

extendArray = (_array, _ext) -> 
	ret = []
	ret.push( val ) for val in _array
	ret.push( val ) for val in _ext
	return ret
#END exendArray

# Method used to write a javascript file
write_file = (filename, body) ->
	body = body.replace(
		/\{\{VERSION\}\}/gi, version
	).replace(
		/\{\{VERSION_TAG\}\}/gi, version_tag
	)
	fs.writeFileSync filename, body
	console.log "Wrote #{filename}"

print_error = (error, file_name, file_contents) ->
	line = error.message.match /line ([0-9]+):/
	if line && line[1] && line = parseInt(line[1])
		contents_lines = file_contents.split "\n"
		first = if line-4 < 0 then 0 else line-4
		last  = if line+3 > contents_lines.size then contents_lines.size else line+3
		console.log "Error compiling #{file_name}. \"#{error.message}\"\n"
		index = 0
		for line in contents_lines[first...last]
			index++
			line_number = first + 1 + index
			console.log "#{(' ' for [0..(3-(line_number.toString().length))]).join('')} #{line}"
	else
		console.log "Error compiling #{file_name}: #{error.message}"

#Task to watch files (so they're built when saved)
task 'watch', 'watch coffee/ and tests/ for changes and build', ->
	console.log "Watching for changes"

	watchers = []

	watch = ->
		console.log("STARTING WATCH ROUTINE")

		try
			build = JSON.parse( "#{fs.readFileSync(build_file)}" ) ? {}
			build["COFFEE"] ?= {}
			build["SASS"] ?= {}
			build["COMBINED"] ?= {}
			build["COPIED"] ?= {}
			build["HAML"] ?= {}

			console.log "STARTING COMPILED COFFEE FILES"
			for d, ss of build["COFFEE"]
				do ->
					destination = d
					ss = [ss] if typeof ss is "string"
					sources = ss

					execute = ->
						code = minified_code = ""
						file_name = file_contents = ""
						
						try
							file_name = destination.replace("{{VERSION}}", version)
							file_contents = ""
							file_contents += "#{fs.readFileSync(source)}\r\n" for source in sources when fs.existsSync(source)

							code = CoffeeScript.compile(file_contents)
							minified_code = UglifyJS.minify(code, {fromString: true}).code
							# minified_code = parser.parse( code )
							# minified_code = uglify.ast_mangle( minified_code )
							# minified_code = uglify.ast_squeeze( minified_code )
							# minified_code = uglify.gen_code( minified_code )

							write_file(file_name, code)
							write_file(file_name.replace(/\.js$/,'.min.js'), minified_code)

							cb() if typeof cb is 'function'
						catch e
							print_error e, file_name, file_contents
						#END try
					#END execute

					for s in sources
						do ->
							source = s
							if fs.existsSync( source )
								console.log "Watching for changes in #{source}"

								watchers.push fs.watch( source, (curr, prev) ->
									console.log "#{new Date}: Saw change in #{source}"
									execute()
								)
							else
								console.error("\r\nERROR: Could not find file '#{source}'' to compile\r\n")
							#END if
						#END do
					#END for

					execute()
				#END do
			#END for

			console.log "STARTING COMPILED SASS FILES"
			count = 0
			temp_prefix = "__temp_" + (new Date).valueOf()

			for d, ss of build["SASS"]
				do ->
					destination = d
					sources = ss

					execute = ->
						try
							file_name = destination.replace("{{VERSION}}", version)
							min_destination = destination.replace(/\.css$/,'.min.css')
							temp_destination = temp_prefix + "_#{count}__.sass"

							file_contents = ""
							file_contents += "#{fs.readFileSync(source)}\r\n" for source in sources when fs.existsSync(source)

							write_file(temp_destination, file_contents)

							exec "sass --update #{temp_destination}:#{destination} --style expanded", (messages) ->
								console.log("Wrote #{destination}")
								console.error( messages ) if messages?.code is 1
								exec "sass --update #{temp_destination}:#{min_destination} --style compressed", ->
									console.log("Wrote #{min_destination}")
									fs.unlink(temp_destination)
									console.log("Cleaning: #{temp_destination}")

									#lastly try to delete in .sass-cache directory
									try
										wrench.rmdirSyncRecursive('.sass-cache')
									catch error
									#END try/catch
								#END min exec
							#END exec

							count++
						catch error
							print_error error, file_name, file_contents
						#END try/catch
					#END return

					for s in sources
						do ->
							source = s
							if fs.existsSync( source )
									console.log "Watching for changes in #{source}"
									watchers.push fs.watch( source, (curr, prev) ->
										console.log "#{new Date}: Saw change in #{source}"
										execute()
									)
								#END do
							else
								console.error("\r\nERROR: Could not find file '#{source}' to compile\r\n")
							#END if
						#END do
					#END for

					execute()
				#END do
			#END for

			console.log "STARTING COMPILED HAML FILES"
			_require = "#{__dirname}/haml/helpers.rb"

			for d, s of build["HAML"]
				do ->
					destination = d
					source = s

					execute = ->
						try
							file_name = destination.replace("{{VERSION}}", version)

							exec "haml --style ugly --double-quote-attributes --no-escape-attrs -r #{_require} --trace #{source} #{file_name}", (messages) ->
								console.log("Wrote #{file_name}")
								print_error( messages ) if messages?
						catch e
							print_error e
						#END try
					#END execute

					if fs.existsSync( source )
						console.log "Watching for changes in #{source}"
						watchers.push fs.watch( source, (curr, prev) ->
							console.log "#{new Date}: Saw change in #{source}"
							execute()
						)
					else
						console.error("\r\nERROR: Could not find file '#{source}' to copy\r\n")
					#END if

					execute()
				#END do
			#END for

			console.log "STARTING COMBINED FILES"
			for d, ss of build["COMBINED"]
				do ->
					destination = d
					sources = ss

					execute = ->
						try
							code = ( fs.readFileSync(source) for source in sources when fs.existsSync(source) ).join("\r\n")
							file_name = destination.replace("{{VERSION}}", version)

							if file_name.indexOf('.min.js') > -1
								code = UglifyJS.minify(code, {
									fromString: true
								}).code
							#END if
							
							write_file( file_name, code )
						catch e
							print_error e, file_name, file_contents
						#END try
					#END execute

					for s in sources
						do ->
							source = s
							if fs.existsSync( source )
								console.log "Watching for changes in #{source}"
								watchers.push fs.watch( source, (curr, prev) ->
									console.log "#{new Date}: Saw change in #{source}"
									execute()
								)
							else
								console.error("\r\nERROR: Could not find file '#{source}' to combine\r\n")
							#END if
						#END do
					#END for

					execute()
				#END do
			#END for

			console.log "STARTING COPIED FILES"
			for d, s of build["COPIED"]
				do ->
					destination = d
					source = s

					execute = ->
						try
							file_name = destination.replace("{{VERSION}}", version)
							fs.createReadStream(source).pipe(fs.createWriteStream(file_name))
							console.log "Wrote #{file_name}"
						catch e
							print_error e
						#END try
					#END execute

					if fs.existsSync( source )
						console.log "Watching for changes in #{source}"
						watchers.push fs.watch( source, (curr, prev) ->
							console.log "#{new Date}: Saw change in #{source}"
							execute()
						)
					else
						console.error("\r\nERROR: Could not find file #{source} to copy\r\n")
					#END if

					execute()
				#END do
			#END for
		catch e
			console.log("COULD NOT COMPILE. ERROR IN BUILD FILE.")
			print_error e
		#END catch
	#END watch

	fs.watch build_file, ->
		console.log("ENDING WATCH ROUTINE")

		watcher.close() for watcher in watchers
		watchers = []

		console.log("REBOOTING")
		watch()
	#END fs.watch

	watch()
#END watch task

