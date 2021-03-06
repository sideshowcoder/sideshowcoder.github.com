# Setup and manage stasis ready github pages directory
#
# provides a couple handy methods for using stasis on github pages to use
# simple copy to a new directory to be used as the base for your github page
#
# rake setup_repository_for_github_user_page
# will initialize the repository as needed
#
# rake publish
# will publish a new version of the site from whatever is generated by stasis
#
# rake development (default so just run rake)
# will run a stasis development server ready on port 3000 locally
#
# Author: Philipp Fehre Github: @sideshowcoder Twitter: @ischi

require 'tmpdir'

desc 'Setup the repository for Github user or organization page run this in an empty folder'
task :setup_repository_for_github_user_page do
  `git init`
  `git checkout -b source`
  write_default_files
  `bundle install`
  `git add -A`
  `git commit -m'setup source branch'`
  `git checkout --orphan master`
  rm_rf FileList['*']
  `touch index.html`
  `git add -A`
  `git commit -m'setup content branch'`
  `git checkout source`
  Rake::Task[:publish].invoke
  puts <<-INFO
Repository setup locally now create a repository named USERNAME.github.com
and point the origin it's way. For more information check the README
  INFO
end

desc 'Generate page'
task :generate do
  generate
end

desc 'Remove generated files from public'
task :clean do
  rm_rf 'public'
end

desc 'Remove generated files including the stasis base layout'
task :deep_clean do
  rm_rf 'public'
  rm FileList['controller.rb', 'Gemfile', 'index.html.erb']
end

desc 'Publish updated page'
task :publish => [:generate] do
  Dir.mktmpdir { |dir|
    mv FileList['public/*'], dir
    current_commit = `git rev-parse --short HEAD`
    `git stash`
    `git checkout master`
    rm_rf FileList['*']
    mv FileList["#{dir}/*"], '.'
    `git add -A`
    `git commit -m 'published from #{current_commit}'`
    `git checkout source`
    `git stash pop`
  }
end

desc 'Start development'
task :development do
  puts 'Starting stasis watcher and server, visit http://localhost:3000 to view'
  pid = fork { exec 'bundle exec stasis -d 3000' }
  begin
    Process.wait pid
  rescue Object
    Process.kill 'TERM', pid
    Process.wait pid
    puts 'Something went wrong, try bundle exec stasis -d 3000 for more information' unless $?.success?
  end
end

task :default => :development

# write some default files to have a valid stasis setup
def write_default_files
  templates = load_templates
  File.open('controller.rb', 'w') { |f| f.write(templates[:controller]) }
  File.open('README.md', 'w') { |f| f.write(templates[:readme]) }
  File.open('index.html.erb', 'w') { |f| f.write(templates[:index]) }
  File.open('Gemfile', 'w') { |f| f.write(templates[:gemfile]) }
end

# generate the public files to serve
def generate
  `bundle exec stasis`
end

# Load embedded templates from the file
# added from Sinatra
def load_templates
  templates = {}
  file = __FILE__

  begin
    io = ::IO.respond_to?(:binread) ? ::IO.binread(file) : ::IO.read(file)
    app, data = io.gsub("\r\n", "\n").split(/^__END__$/, 2)
  rescue Errno::ENOENT
    app, data = nil
  end

  if data
    lines = app.count("\n") + 1
    template = nil
    data.each_line do |line|
      lines += 1
      if line =~ /^@@\s*(.*\S)\s*$/
        template = ''
        templates[$1.to_sym] = template
      elsif template
        template << line
      end
    end
  end

  templates
end

__END__

@@ gemfile
source "https://rubygems.org"

gem "stasis"
gem "redcarpet"

@@ controller
# ignore everything needed to build
ignore 'Gemfile.lock'
ignore 'Gemfile'
ignore 'Rakefile'
# ignore all the git stuff
ignore '.gitignore'
ignore '.ruby-version'
ignore '.git'
# we don't want to render the readme
ignore 'README.md'

@@ index
<!DOCTYPE html>
<html>
<head>
  <title>My Github user page</title>
</head>
<body>
<div class="container">

  <h1>My Github user page</h1>
  <p>checkout <a href='http://stasis.me'>stasis<a> to know how to work with me. To publish just run</p>
  <code>rake publish<code>

</div>
</body>
</html>

@@ readme
Repository setup locally now create a repository named USERNAME.github.com
and point the origin it's way.

The rest of the Rakefile assumes a stasis default setup to be used so

    rake development
runs a local development server on port 3000, watching the current dir

    rake generate
generates the current page inside public

    rake publish
publishes whatever is currently generated to the master branch ready to push

Have a great day!
@sideshowcoder <Twitter @ischi>
