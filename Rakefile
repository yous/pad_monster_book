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
  dictionary = File.open(asset_file('dictionary.txt'), 'rb') { |f| f.read }
  if response == dictionary
    puts 'Already up-to-date.'
    exit
  end
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

desc 'Generate grayscale icons'
task :grayscale do
  require 'yaml'
  require 'RMagick'

  # Read from sort.yml to get the list of grayscale monster.
  grayscale_monster = []
  monster_sort = YAML.load(File.read(asset_file('sort.yml')))
  monster_sort.values.flatten(1).each do |monster_row|
    # -
    next unless monster_row
    # - grayscale: true
    #   items: [122, 124, 126, 128, 130]
    if monster_row['grayscale']
      monster_row['items'].each do |monster_id|
        next if File.exist?(asset_file("icons/grayscale/#{monster_id}.png"))
        grayscale_monster << monster_id
      end
    # - items:
    #   - grayscale: true
    #     item: 648
    elsif !monster_row['combined']
      monster_row['items'].select { |x| x.is_a?(Hash) && x['grayscale'] }
        .each do |monster|
        next if File.exist?(
          asset_file("icons/grayscale/#{monster['item']}.png"))
        grayscale_monster << monster['item']
      end
    end
  end

  if grayscale_monster.empty?
    puts 'Already up-to-date.'
    exit
  end
  puts 'Generating grayscale icons...'
  grayscale_monster.each do |monster_id|
    image = Magick::ImageList.new(asset_file("icons/#{monster_id}.png"))
    grayscale_image = image.cur_image.quantize(256, Magick::GRAYColorspace,
                                               Magick::NoDitherMethod)
    grayscale_image.write(asset_file("icons/grayscale/#{monster_id}.png"))
  end
  puts 'Done.'
end

desc 'Generate combined icons'
task :combine do
  require 'yaml'
  require 'RMagick'

  # Read from sort.yml to get the list of combined icon.
  combined_icons = []
  monster_sort = YAML.load(File.read(asset_file('sort.yml')))
  monster_sort.values.flatten(1).each do |monster_row|
    # -
    next unless monster_row
    # - combined: true
    #   items:
    #   - items: [388, 389]
    #   - items: [390, 391]
    if monster_row['combined']
      monster_row['items'].each do |combined|
        next if File.exist?(
          asset_file("icons/combined/#{combined['items'].join('_')}.png"))
        combined_icons << combined['items']
      end
    # - items:
    #   - combined: true
    #     items: [755, 756]
    elsif !monster_row['grayscale']
      monster_row['items'].select { |x| x.is_a?(Hash) && x['combined'] }
        .each do |monster|
        next if File.exist?(
          asset_file("icons/combined/#{monster['items'].join('_')}.png"))
        combined_icons << monster['items']
      end
    end
  end

  if combined_icons.empty?
    puts 'Already up-to-date.'
    exit
  end
  puts 'Generating combined icons...'
  combined_icons.each do |monster_ids|
    above_id, below_id = monster_ids
    above_image = Magick::ImageList.new(asset_file("icons/#{above_id}.png"))
    below_image = Magick::ImageList.new(asset_file("icons/#{below_id}.png"))

    unless above_image.columns == below_image.columns &&
      above_image.rows == below_image.rows
      warn 'Images have different size: ' \
        "#{above_id}: #{above_image.columns}x#{above_image.rows}, " \
        "#{below_id}: #{below_image.columns}x#{below_image.rows}"
      exit
    end

    above_image.rows.times do |y|
      above_image.columns.times.select { |x| x <= y }.each do |x|
        above_image.pixel_color(x, y, below_image.pixel_color(x, y))
      end
    end
    above_image.write(asset_file("icons/combined/#{monster_ids.join('_')}.png"))
  end
  puts 'Done.'
end

require 'rubocop/rake_task'
RuboCop::RakeTask.new(:rubocop)

task default: :rubocop
