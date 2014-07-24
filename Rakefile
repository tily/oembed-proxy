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
	
		match_data = paragraph.text.match(/\s*(?<name>.+) \((?<url>.+)\)/)
		provider['name'], provider['url'] = match_data[:name], match_data[:url]
	
		list = paragraph.xpath('following::ul[1]/li')
		list.each do |item|
			case item.text
			when /URL (s|S)cheme( \(videos\))?:\s+([^\s]+)/
				provider['url_scheme'] << $3
			when /(API endpoint|Endpoint):\s+([^\s]+)/
				provider['api_endpoint'] << $2
			when /Documentation:\s+([^\s]+)/
				provider['documentation'] << $1
			when /Example:\s+([^\s]+)/
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

		case provider['name']
		when 'Ted'
			provider['api_endpoint'] = 'http://www.ted.com/talks/oembed.{format}'
		when 'Jest'
			provider['url_scheme'] = 'http://www.jest.com/video/*'
		when 'YouTube'
			provider['url_scheme'] = 'http://www.youtube.com/watch?*'
		when 'Vimeo'
			provider['url_scheme'] = 'http://vimeo.com/*'
		end

		if provider['api_endpoint'].empty?
			puts "Warning: #{provider['name']} does not include api_endpoint"
		end

		if provider['url_scheme'].empty?
			puts "Warning: #{provider['name']} does not include url_scheme"
		end

		if provider['name'] == 'Embedly'
			next
		end
	
		providers << provider
	end
	
	File.write('./data/providers.json', JSON.pretty_generate(providers))
end

desc 'scrape microsoft mime types'
task :scrape_microsoft_mime_types do
	doc = Nokogiri::HTML(open('http://filext.com/faq/office_mime_types.php'))
	cells = doc.xpath('//table[@class="MsoNormalTable"]/tbody/tr/td[2]')
	types = cells.reject {|c| c.text == 'MIME Type' }.map {|c| c.text }.uniq
	File.write('./data/microsoft_mime_types.json', JSON.pretty_generate(types))
end

require 'httparty'
require 'json'
desc 'try embedly'
task :embedly do
	key = ENV['EMBEDLY_API_KEY']
	target_url = ENV['URL']
	url = "http://api.embed.ly/1/oembed?key=#{key}&url=#{target_url}&format=json"
	puts JSON.pretty_generate JSON.parse HTTParty.get(url).body
end
