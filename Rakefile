require 'json'
require 'bundler/gem_tasks'
require 'pry-debugger'

desc 'remove SCSS files'
task :clean_scss do
  rm_rf 'app/assets/*'
  puts "cleaned old SCSS files"
  puts "-----"
end

desc 'move jquery-ui-rails stylesheets'
task :move_stylesheets do
  target_dir = 'jquery-ui-rails/stylesheets'
  mkdir_p target_dir
  system("mv app/assets/stylesheets/*.erb #{target_dir}")
  puts "move stylesheets to #{target_dir} (if necesarry)"
  puts "-----"
end

desc "Convert stylesheets to SCSS"
task :convert_to_scss do
  source_dir = 'jquery-ui-rails/stylesheets'
  target_dir = 'app/assets/stylesheets'
  mkdir_p target_dir
  variables_hash = {}

  # convert css files to scss
  Dir.glob("#{source_dir}/*.erb").each do |source_file|
    puts source_file
    stylesheet_content = File.read(source_file)

    # replace image_path ERB with image-url Sass
    stylesheet_content.gsub!(/<%= image_path\((\S+)\) %>/) { "image-path(#{$1})" }

    # remove comment blocks with sprockets require directives, because they don't work well with Sass variables
    stylesheet_content.gsub!(/\/\*[^\/]+require[^\/]+\*\//) do |match|
      if source_file.end_with?('jquery.ui.theme.css.erb')
        "@import 'themes/jquery.ui.base';\n"

      elsif source_file.end_with?('jquery.ui.all.css.erb') || source_file.end_with?('jquery.ui.base.css.erb')
        match.gsub!(/ \*= require ([a-z\.]+)/) { "@import '#{$1}';" }
        match.gsub!(/\/\*| \*\//, '')

      else
        ''
      end
    end

    # extract vars
    regex = /(url\(<%= image_path\([\S]+\) %>\)|[\S]+)\/\*{([a-z]+)}\*\//i
    vars = stylesheet_content.scan regex

    # write variables to gobal hash
    vars.each do |var|
      value = var[0]
      name = var[1]
      variables_hash[name] ||= value
    end

    # write SCSS file
    destination_file_name = File.basename(source_file).gsub(".css.erb", ".css.scss")
    destination_file = File.open "#{target_dir}/#{destination_file_name}", 'w'
    destination_file << stylesheet_content.gsub(regex) { "$#{$2}" }
    destination_file.close
    puts "> .css.scss"
    puts "--"
  end

  # write _jquery.ui.base.css.scss
  mkdir_p "#{target_dir}/themes"
  variables_stylesheet = File.open "#{target_dir}/themes/_jquery.ui.base.css.scss", 'w'
  variables_hash.each do |name, value|
    variables_stylesheet << "$#{name}: #{value} !default;\n"
  end
  variables_stylesheet.close
  puts "wrote _jquery.ui.base.css.scss"
  puts "-----"
end


require 'open-uri'
task :themes do
  theme_js = ""
  open("https://raw.github.com/jquery/download.jqueryui.com/master/lib/themeroller.themegallery.js") {|f|
    theme_js = f.read
  }

  theme_js.scan(/vars: ({[^}]+})/) do |match|
    theme_string = match[0]
    theme_string.gsub! 'name', '"name"' # turn into parseable JSON

    theme = JSON.parse theme_string

    theme_name = theme.delete('name').downcase.gsub(' ', '-')

    puts theme.inspect

    target_dir = 'app/assets/stylesheets/themes'
    mkdir_p target_dir
    theme_stylesheet = File.open "#{target_dir}/_jquery.ui.#{theme_name}.css.scss", 'w'
    theme.each do |name, value|
      theme_stylesheet << "$#{name}: #{value} !default;\n"
    end
    theme_stylesheet.close
  end
end

task :scss_process => [:clean_scss, :move_stylesheets, :convert_to_scss, :themes]

task :default => :scss_process
