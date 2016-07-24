require 'middleman-gh-pages'
require 'json'
require 'bundler'

desc 'Generates all the static resources necessary to run the site.'
task generate: ['generator:plugins_core', 'generator:plugins_external', 'generator:grab_dangerfiles', 'generator:danger_search_plugin_json'] do
  # Your Gemfile.lock tends to get put out of sync after some of the commands.
  puts 'Shippit'
end

desc 'Things that generate JSON'
namespace :generator do
  desc 'Generates a JSON file that represents the core plugins documentation'
  task :plugins_core do
    # Make sure we have a folder for json_data, it's not in git.
    Dir.mkdir('static/json_data') unless Dir.exist?('static/json_data')

    # Grab the Danger gem, pull out the file paths for the core plugins
    danger_gem = Gem::Specification.find_by_name 'danger'
    danger_core_plugins = Dir.glob(danger_gem.gem_dir + '/lib/danger/danger_core/plugins/*.rb')

    # Document them, move them into a nice JSON file
    output = `bundle exec danger plugins json #{danger_core_plugins.join(' ')}`
    abort('Could not generate the core plugin metadata') if output.empty?
    File.write('static/json_data/core.json', output)
    puts 'Generated core API metadata'
  end

  desc 'Generates a JSON file that represents the external plugins documentation'
  task :plugins_external do
    plugins = JSON.parse(File.read('plugins.json'))

    # Generate the Website's plugin doc, by passing in the gem names
    output = `bundle exec danger plugins json #{plugins.join(' ')}`
    abort('Could not generate any plugin metadata') if output.empty?
    File.write('static/json_data/plugins.json', output)
    puts 'Generated plugin metadata'
  end

  desc 'Generates a JSON file that represents the external plugins documentation'
  task :grab_dangerfiles do
    # Grab our Dangerfile plugins list
    dangerfile_repos = JSON.parse(File.read('example_oss_dangerfiles.json'))
    dangerfile_repos.each do |repo|
      require 'open-uri'
      require 'pygments'
      branch = repo.include?('#') ? repo.split('#').last : 'master'
      repo = repo.split('#').first

      dangerfile = open("https://raw.githubusercontent.com/#{repo}/#{branch}/Dangerfile").read

      path = "static/source/dangerfiles/#{repo.tr('/', '_')}.html"
      html = Pygments.highlight(dangerfile, lexer: 'ruby', options: { encoding: 'utf-8' })

      File.write(path, html)
    end
    puts 'Downloaded Dangerfiles to `static/source/dangerfiles`'
  end

  desc 'Generate the search plugin JSON file, this used by `danger plugins search` and by the re-deploy webhook system.'
  task :danger_search_plugin_json do
    # Grab our definitive plugins list of
    plugins = JSON.parse(File.read('plugins.json'))

    Bundler.with_clean_env do
      Dir.mktmpdir do |dir|
        Dir.chdir dir do
          gemfile = File.new('Gemfile', 'w')
          gemfile.write "source 'https://rubygems.org'\n"

          plugins.each do |plugin|
            gemfile.write "gem '#{plugin}'\n"
          end
          gemfile.close

          `bundle install --path vendor/gems`

          spec_paths = plugins.flat_map { |plugin| Dir.glob("vendor/gems/ruby/*/specifications/#{plugin}*.gemspec").first }
          real_gems = spec_paths.map { |path| Gem::Specification.load path }

          plugin_metadata = real_gems.map do |gem|
            {
              gem: gem.name,
              authors: gem.authors.join(', '),
              url: gem.homepage,
              description: gem.summary,
              license: gem.license || 'Unknown',
              version: gem.version.to_s
            }
          end

          plugin_json = { plugins: plugin_metadata }
          plugins_file_path = File.join File.dirname(__FILE__), 'plugins-search-generated.json'
          File.write(plugins_file_path, plugin_json.to_json)
          puts 'Generated search metadata'
        end
      end
    end
  end

  desc 'Generate the website plugin search JSON file, this is different from the danger gem search - as one gem can have multiple plugins'
  task :danger_search_plugin_json do
    # TODO
  end
end

desc 'Runs the site locally'
task :serve do
  puts 'Running locally at http://localhost:4567'
  sh 'open http://localhost:4567'
  Dir.chdir('static') do
    sh 'bundle exec middleman server'
  end
end
