# frozen_string_literal: true

source "https://rubygems.org"

# 固定版本：站点含基于 7.0.1 定制的 _layouts/post.html 覆盖（tail_includes 引用了 comments 等 7.0.x include），
# 升级主题时必须同步重做覆盖并完整回归后再放开版本约束（7.6.0 已将 comments 改为 script_includes: [comment]）。
gem "jekyll-theme-chirpy", "= 7.0.1"

group :test do
  gem "html-proofer", "~> 5.0"
end
