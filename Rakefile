# encoding: utf-8

require 'rubygems'
require 'bundler'
begin
  Bundler.setup(:default, :development)
rescue Bundler::BundlerError => e
  $stderr.puts e.message
  $stderr.puts "Run `bundle install` to install missing gems"
  exit e.status_code
end
require 'rake'

require 'jeweler'
Jeweler::Tasks.new do |gem|
  # gem is a Gem::Specification... see http://guides.rubygems.org/specification-reference/ for more options
  gem.name = "oembed-proxy"
  gem.homepage = "http://github.com/tily/oembed-proxy"
  gem.license = "MIT"
  gem.summary = %Q{TODO: one-line summary of your gem}
  gem.description = %Q{TODO: longer description of your gem}
  gem.email = "tily05@gmail.com"
  gem.authors = ["tily"]
  # dependencies defined in Gemfile
end
Jeweler::RubygemsDotOrgTasks.new

require 'rspec/core'
require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new(:spec) do |spec|
  spec.pattern = FileList['spec/**/*_spec.rb']
end

desc "Code coverage detail"
task :simplecov do
  ENV['COVERAGE'] = "true"
  Rake::Task['spec'].execute
end

task :default => :spec

require 'rdoc/task'
Rake::RDocTask.new do |rdoc|
  version = File.exist?('VERSION') ? File.read('VERSION') : ""

  rdoc.rdoc_dir = 'rdoc'
  rdoc.title = "oembed-proxy #{version}"
  rdoc.rdoc_files.include('README*')
  rdoc.rdoc_files.include('lib/**/*.rb')
end

require 'json'
require 'nokogiri'
require 'open-uri'

desc 'scrape oembed providers and save to data/providers.json'
task :scrape_oembed_providers do
	providers = []
	
	doc = Nokogiri::HTML(open('http://oembed.com'))
	
	paragraphs = (doc.xpath('//a[@id="section7.1"]/following::p') & doc.xpath('//a[@id="section7.2"]/preceding::p'))
	
	paragraphs.each do |paragraph|
		provider = Hash.new {|h, k| h[k] = Array.new }
	
		match_data = paragraph.text.match(/(?<name>.+) \((?<url>.+)\)/)
		provider['name'], provider['url'] = match_data[:name], match_data[:url]
	
		list = paragraph.xpath('following::ul[1]/li')
		list.each do |item|
			case item.text
			when /URL scheme: ([^\s]+)/
				provider['url_scheme'] << $1
			when /API endpoint: ([^\s]+)/
				provider['api_endpoint'] << $1
			when /Documentation: ([^\s]+)/
				provider['documentation'] << $1
			when /Example: ([^\s]+)/
				provider['example'] << $1
			when /Supports discovery via <link> tags/
				provider['supports_discovery_via_link_tags'] = true
			end
		end
	
		provider.each do |key, value|
			if value.is_a?(Array) && value.size == 1
				provider[key] = value.first
			end
		end
	
		providers << provider
	end
	
	File.write('./data/providers.json', JSON.pretty_generate(providers))
end
