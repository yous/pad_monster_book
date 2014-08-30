# encoding: utf-8

desc 'Update the monster dictionary file by getting from puzzledragonx.com'
task :update do
  require 'net/http'
  url = URI.parse(
    'http://www.puzzledragonx.com/en/script/autocomplete/dictionary.txt')
  req = Net::HTTP::Get.new(url.to_s)
  puts 'Getting the dictionary file from puzzledragonx.com...'
  res = Net::HTTP.start(url.host, url.port) { |http| http.request(req) }
  dictionary_file = File.expand_path('../assets/dictionary.txt', __FILE__)
  File.open(dictionary_file, 'wb') { |f| f.write(res.body) }
  puts 'Done.'
end
