require 'tmpdir'

desc 'Setup the repository for Github user or organization page'
task :setup_repository_for_github_user_page do
end

desc 'Generate page'
task :generate do
  `bundle exec stasis`
end

desc 'Remove generated files'
task :clean do
  rm_rf 'public'
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
    `git commit -m 'published from #{current_commit}`
    `git push origin master`
    `git checkout source`
    `git stash pop`
  }
end

desc 'Start development processes'
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

