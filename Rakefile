require 'bundler'
require 'ruby-maven'
require 'rake/testtask'

Bundler::GemHelper.install_tasks

Rake::TestTask.new do |t|
  t.pattern = "test/*_test.rb"
end

desc "Run benchmarks"
task :benchmark do
  load 'benchmarking/benchmark_threaded.rb'
end

desc "Pack jar after compiling classes, use this to rebuild the pom.xml"
task :compile do
  RubyMaven.exec('prepare-package')
  # after packaging the jrjackson-x.y.z.jar vendor jar dependencies
  Rake::Task['vendor_jars'].invoke
  Rake::Task['verify_generated_files'].invoke
end

task :vendor_jars do
  require 'jars/installer'
  Jars::Installer.vendor_jars!
end

# Fail the build if a file the gemspec lists is missing on disk. This stops a
# broken gem from shipping, as happened with 0.5.1.
task :verify_generated_files do
  $LOAD_PATH.unshift(File.expand_path('lib', __dir__))
  require 'jrjackson/build_info'
  missing = JrJackson::BuildInfo.files.reject { |f| File.file?(f) }
  unless missing.empty?
    raise "These files are listed in the gemspec but missing after compile #{missing.join(', ')}"
  end
end

desc "Clean build"
task :clean do
  RubyMaven.exec('clean')
end

task :default => [ :compile ]

require 'rubygems/package_task'
Gem::PackageTask.new( eval File.read( 'jrjackson.gemspec' ) ) do
  desc 'Pack gem'
  task :package => [:compile]
end

# The logstash-plugins/publisher action runs `rake vendor`. For jrjackson that
# means the Maven build that generates the jar and lib/jrjackson_jars.rb, which
# compile already does (and which runs verify_generated_files).
desc "Build native artifacts (alias of compile, used by the publisher action)"
task :vendor => :compile

# The logstash-plugins/publisher "Gem Publish" action defaults to `rake release`.
# Replace Bundler's release task, which does a bare `gem build` with no Maven
# compile, so releasing needs no custom task argument. Build through :package so
# the gem is compiled and verify_generated_files runs before we push. Uses the
# exact version filename, not a glob, so a stale pkg/ gem cannot be pushed by
# mistake.
Rake::Task[:release].clear
desc "Build the gem, push to RubyGems, then tag the repo"
task :release => :package do
  $LOAD_PATH.unshift(File.expand_path('lib', __dir__))
  require 'jrjackson/build_info'
  version = JrJackson::BuildInfo.version
  gem_file = "pkg/jrjackson-#{version}-java.gem"
  raise "Built gem #{gem_file} not found" unless File.file?(gem_file)
  sh "gem push #{gem_file}"
  sh "git tag v#{version}"
  sh "git push origin v#{version}"
end

# Backwards-compatible alias for the previous task name.
task :publish_gem => :release
