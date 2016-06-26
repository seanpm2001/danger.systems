desc "Runs the site locally"
task :serve do
  puts 'Running locally at http://localhost:4567'
  sh "open http://localhost:4567"
  sh "bundle exec middleman server"
end

task :generate do
  puts "TODO"
end

desc "Ship to github pages"
begin
  require 'middleman-gh-pages'
  desc 'Build and push the guides to GitHub Pages'
  task :deploy do
    Rake::Task['generate'].invoke
    system("rake publish PROJECT_ROOT='static'")
  end
rescue LoadError
  $stderr.puts "[!] Disabled the middleman publish task, run `bundle exec` first."
end