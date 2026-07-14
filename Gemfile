# Pre-patch Ruby 4.0 untaint deprecation for older Liquid versions
class String; def untaint; self; end; end

source "https://rubygems.org"

# We use modern versions to stay fully compatible with your Ruby 4.0.5 setup
gem "jekyll", "~> 4.3"
gem "webrick"
gem "csv"
gem "bigdecimal"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-sitemap"
  gem "jekyll-seo-tag"
  gem "minima"
end
