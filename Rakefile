require "bundler/gem_tasks"

# To find which jar file contains a java class
# find tmp -name *.jar -exec bash -c "echo {} && jar tvf {} | grep StaticLoggerBinder " \;

def download_folder
  abort "Please create a #{File.expand_path('tmp')} folder and copy the neo4j enterprise gz/tar file downloaded from http://neo4j.org/download" unless File.directory?('tmp')
  Dir.new('tmp')
end

def tar_file
  download_folder.entries.find { |x| x =~ /gz$/ || x =~ /tar$/}.tap do |f|
    abort "expected a neo4j .gz/.tar file in folder #{File.expand_path(download_folder.path)}"  unless f
  end
end

def source_file
  File.expand_path("./tmp/#{tar_file}")
end

def unpack_lib_dir
  dir = tar_file.gsub('-unix.tar.gz', '')
  dir = dir.gsub('-unix.tar', '')
  File.expand_path("./tmp/#{dir}/lib")
end

def unpack_system_lib_dir
  dir = tar_file.gsub('-unix.tar.gz', '')
  dir = dir.gsub('-unix.tar', '')
  File.expand_path("./tmp/#{dir}/system/lib")
end

def jar_files_to_copy
  Dir.new(unpack_lib_dir).entries.find_all {|x| x =~ /\.jar/}
end

def system_jar_files_to_copy
  Dir.new(unpack_system_lib_dir).entries.find_all {|x| x =~ /\.jar/}  
end

desc "Delete old Jar files"
task :delete_old_jar do
  root = File.expand_path("./lib/neo4j-enterprise/jars")
  files = Dir.new(root).entries.find_all{|f| f =~ /\.jar/}
  files.each do |file|
    system "git rm #{root}/#{file}" unless file.start_with?('log4j')
  end
end

def include_jar?(file)
  include_only = %w[server-api neo4j-udc neo4j-consistency-check log4j logback-core zookeeper logback-classic neo4j-cluster netty neo4j-backup neo4j-com neo4j-enterprise neo4j-ha org.apache.servicemix.bundles.netty slf4j]
  include_only.each do |i|
    return true if file.start_with?(i) 
  end
  false
end

desc "Upgrade using downloaded ...tar.gz file in ./tmp"
task :upgrade => [:delete_old_jar] do
  system "cd tmp; tar xf #{source_file}"
  jars = File.expand_path("./lib/neo4j-enterprise/jars")
  system "mkdir -p #{jars}"
  jar_files_to_copy.each do |f| 
    next unless include_jar?(f)
    system "cp #{unpack_lib_dir}/#{f} #{jars}/"
    system "git add -f #{jars}/#{f}"
  end

  system_jar_files_to_copy.each do |f| 
    next unless include_jar?(f)
    system "cp #{unpack_system_lib_dir}/#{f} #{jars}/"
    system "git add -f #{jars}/#{f}"
  end
end