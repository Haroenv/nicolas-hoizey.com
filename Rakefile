require "rubygems"
require "tmpdir"
require "bundler/setup"
require "jekyll"
require "algoliasearch"

namespace :site do
  jekyll_config = Jekyll.configuration(source: ".", destination: "_site")
  jekyll_credentials = YAML.load_file("./_config_credentials.yml")
  jekyll_config = jekyll_config.merge jekyll_credentials
  jekyll_site = Jekyll::Site.new(jekyll_config)

  desc "Generate blog files"
  task :generate do
    jekyll_site.process
  end

  desc "Generate and index blog"
  task :index, [:algolia_api_key] => :generate do |t, args|
    raise "missing algolia_api_key argument" if args[:algolia_api_key].nil?

    # send all title/urls to Algolia's indexing API
    Algolia.init application_id: jekyll_config['algolia']['application_id'], api_key: args[:algolia_api_key]
    index = Algolia::Index.new(jekyll_config['algolia']['index_name'])

    index.set_settings attributesToIndex: ['title', 'tags', 'content', 'unordered(url)', 'unordered(year)']
    index.clear! rescue "not fatal"
    index.add_objects jekyll_site.posts.map { |post| {
      title: post.title,
      tags: post.tags,
      date: post.date,
      content: post.content.gsub(/<[^>]*>/ui, '').gsub(/<!--(.*?)-->[\n]?/m, ""),
      url: post.url,
      year: post.date.strftime("%Y")
    } }
  end
end
