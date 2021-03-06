require 'rake'
require 'ftools' if RUBY_VERSION < "1.9"

home = `printf $HOME`
@timestamp = Time.now.strftime("%Y-%m-%d_%H-%M-%S")
@replace_all = false
NUM = 75

task :default => 'install:all'

def replace_file(filename)
  dest_file = homepath(filename)
  if (File.exist? dest_file) && !(File.symlink? dest_file)
    if @replace_all
      replace(filename)
    else
      print "Overwrite #{dest_file}? [ynaq] "
      case $stdin.gets.chomp
      when 'a'
        puts "Replacing all files..."
        @replace_all = true
        replace(filename)
      when 'y'
        replace(filename)
      when 'q'
        exit
      else
        puts "Skipping #{dest_file}"
      end
    end
  else
    replace(filename)
  end
end

def backup_file(filename)
  puts "Backing up $HOME/#{filename} to $HOME/_dot_backups/#{@timestamp}/#{filename}"
  system %Q{mkdir -p "$HOME/_dot_backups/#{@timestamp}"}
  system %Q{cp -Rf "$HOME/#{filename}" "$HOME/_dot_backups/#{@timestamp}/#{filename}"}
end

def replace(filename)
  filepath = homepath(filename)
  if (File.exist? filepath) && !(File.symlink? filepath)
    center_string "#{filepath} is a regular file, replacing."
    backup_file(filename)
    remove_file(filepath)
    link_file(filename)
  elsif (File.symlink? filepath)
    center_string "#{filepath} is a symlink, replacing."
    remove_file(filepath)
    link_file(filename)
  elsif !(File.exist? filepath)
    center_string "#{filepath} doesn't exist, linking."
    link_file(filename)
  end
end

def link_file(filename)
  system %Q{ln -s "#{currentpath(filename.gsub(/^./, ""))}" "#{homepath(filename)}"}
end

def remove_file(filepath)
   puts "rm -rf #{filepath}"
   system %Q{rm -rf #{filepath}}
end

def homepath(filename)
  File.join(ENV['HOME'], filename)
end

def currentpath(filename)
  File.join(ENV['PWD'], filename)
end

def divider
  puts
  NUM.times do print "=" end
  puts
end

def center_string(string)
  puts string.center(NUM)
end

namespace :install do
  task :all do
     Rake::Task['install:files'].invoke
  end

  desc "install the dot files into user's home directory"
  task :files do
    files = Dir['*']

    files.each do |file|
      next if %w[Rakefile README LICENSE bin].include? file
      divider if file == files.first
      puts
      replace_file("."+file)
      divider
      puts if file == files.last
    end
  end

  desc "Create symbolic link for kaleidoscope integration with git difftool"
  task :ksdiff do
    ksdiff = File.expand_path("/usr/local/bin/ksdiff-wrapper")
    opendiff = File.expand_path("/usr/bin/opendiff")
    if File.exist?(ksdiff)
      if !File.exist?(opendiff) && !File.symlink?(opendiff)
        puts "#{opendiff} doesn't exist, linking..."
        system %Q{sudo ln -s #{ksdiff} #{opendiff}}
      elsif File.exist?(opendiff) && !File.symlink?(opendiff)
        # file already exists. back it up
        puts "moving #{opendiff} to #{opendiff}_orig"
        system %Q{sudo mv #{opendiff} #{opendiff}_orig}
        puts "linking #{ksdiff} to #{opendiff}"
        system %Q{sudo ln -s #{ksdiff} #{opendiff}}
      else File.exist?(opendiff) && File.symlink?(opendiff)
        puts "file already linked"
      end
    else
      puts "please install ksdiff before running this tool"
    end
  end
end
