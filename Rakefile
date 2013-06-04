#!/usr/bin/env rake

POST_TEMPLATE = '_templates/default_post.txt'

namespace :blog do
  desc "Create a new blog post with a custom title"
  task :new, :title do |t, params|
    title = params.title.downcase.strip

    title.gsub! /['`"]/, ""
    title.gsub! /\s*@\s*/, " at "
    title.gsub! /\s*&\s*/, " and "
    title.gsub! /\s*[^A-Za-z0-9\.\-]\s*/, '-'
    title.gsub! /-+/,"-"
    title.gsub! /\A[-\.]+|[-\.]+\z/, ""

    title.insert 0, Time.now.strftime("%Y-%m-%d-")
    title.insert -1, '.md'

    rendered_post = File.read(POST_TEMPLATE).gsub /{{title}}/, params.title

    File.open('_posts/' + title, 'w') {|f| f.write(rendered_post) }

    puts "Created " + title
  end
end

def git_clean?
  git_state = `git status 2> /dev/null | tail -n1`
  clean = (git_state =~ /working directory clean/)
end

task :check_git do
  unless git_clean?
    puts "Dirty repo - commit or discard your changes and run deploy again"
    exit 1
  end
end

desc "Deploy to remote origin"
task :deploy => [:check_git] do
  source_branch = 'source'
  deploy_branch = 'master'
  message = "Site updated at #{Time.now.utc}"

  system "bundle exec jekyll build"
  system "git checkout \"#{deploy_branch}\""
  system "cp -r _site/* . && rm -rf _site/ && touch .nojekyll"

  unless git_clean?
    system "git add . && git commit -m \"#{message}\""
    system "git push origin \"#{deploy_branch}\""
    puts "Pushed to origin and pages with commit message: #{message}"
  else
    puts "No changes to deploy - canceled"
  end

  system "git checkout \"#{source_branch}\""
end