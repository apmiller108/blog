source "https://rubygems.org"
ruby RUBY_VERSION

require 'json'
require 'open-uri'

versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
gem "minima", "~> 2.0"
gem 'jekyll-seo-tag'

group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
end
