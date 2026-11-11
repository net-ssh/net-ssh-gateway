require "bundler/gem_tasks"
require "rake/testtask"

Rake::TestTask.new do |t|
  t.libs = ["lib", "test"]
  t.pattern = "test/**/*_test.rb"
end

task :default => :test

namespace :cert do
  desc "Update public cert from private - only run if public is expired"
  task :update_public_when_expired do
    require 'openssl'
    require 'time'
    raw = File.read "net-ssh-public_cert.pem"
    certificate = OpenSSL::X509::Certificate.new raw
    raise Exception, "Not yet expired: #{certificate.not_after}" unless certificate.not_after < Time.now
    sh "gem cert --build netssh@solutious.com --days 365*5 --private-key /mnt/gem/net-ssh-private_key.pem"
    sh "mv gem-public_cert.pem net-ssh-public_cert.pem"
    sh "gem cert --add net-ssh-public_cert.pem"
  end
end

VERSION_FILE = "lib/net/ssh/gateway/version.rb"

# Sets the version in VERSION_FILE and turns the "=== Unreleased" section of CHANGES.txt into its release entry.
def change_version
  source = File.read(VERSION_FILE)
  current = source[/VERSION = "(\d+\.\d+\.\d+)"/, 1] or raise "No VERSION in #{VERSION_FILE}"
  new_version = yield(*current.split(".").map(&:to_i)).join(".")

  changes = File.read("CHANGES.txt")
  raise "CHANGES.txt has no '=== Unreleased' section" unless changes.start_with?("=== Unreleased\n")

  File.write(VERSION_FILE, source.sub(%("#{current}"), %("#{new_version}")))
  File.write("CHANGES.txt", changes.sub("=== Unreleased", "=== #{new_version} / #{Time.now.utc.strftime('%d %b %Y')}"))
  puts "#{current} => #{new_version}"
end

namespace :vbump do
  desc "Patch release"
  task :final do
    change_version { |major, minor, tiny| [major, minor, tiny + 1] }
  end

  desc "Minor release"
  task :minor do
    change_version { |major, minor, _tiny| [major, minor + 1, 0] }
  end
end
