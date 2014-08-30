# encoding: utf-8

require 'json'
require 'net/http'

def asset_file(file_name)
  File.expand_path("../assets/#{file_name}", __FILE__)
end

def http_get(url)
  url = URI.parse(url)
  req = Net::HTTP::Get.new(url.to_s)
  response = Net::HTTP.start(url.host, url.port) { |http| http.request(req) }
  response.body
end

desc 'Update the monster dictionary file by getting from puzzledragonx.com'
task :update do
  puts 'Getting the dictionary file from puzzledragonx.com...'
  response = http_get(
    'http://www.puzzledragonx.com/en/script/autocomplete/dictionary.txt')
  File.open(asset_file('dictionary.txt'), 'wb') { |f| f.write(response) }
  puts 'Done.'
end

desc 'Download the monster icons from puzzledragonx.com'
task :download do
  dictionary = JSON.parse(File.read(asset_file('dictionary.txt')))
  dictionary['monsters'].map { |monster| monster[0] }.each do |monster_id|
    icon_file = asset_file("icons/#{monster_id}.png")
    next if File.exist?(icon_file)
    puts "Getting the icon of monster No.#{monster_id}..."
    response = http_get(
      "http://www.puzzledragonx.com/en/img/book/#{monster_id}.png")
    File.open(icon_file, 'wb') { |f| f.write(response) }
  end
  puts 'Done.'
end

require 'rubocop/rake_task'
RuboCop::RakeTask.new(:rubocop)

task default: :rubocop
