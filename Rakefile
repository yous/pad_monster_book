# encoding: utf-8

require 'json'
require 'yaml'
require 'net/http'
require 'RMagick'

def asset_file(file_name)
  File.expand_path("../assets/#{file_name}", __FILE__)
end

# Load constants from YAML files.
SORT    = YAML.load(File.read(asset_file('sort.yml')))
MAP     = YAML.load(File.read(asset_file('map.yml')))
ARTICLE = YAML.load(File.read(asset_file('article.yml')))

def http_get(url)
  url = URI.parse(url)
  req = Net::HTTP::Get.new(url.to_s)
  response = Net::HTTP.start(url.host, url.port) { |http| http.request(req) }
  response.body
end

def monster_link(monster_id, grayscale: false, combined: false)
  if combined
    %(<img src="#{monster_icon_link(monster_id, combined: true)}">)
  else
    '<a class="tig-tooltip tig-tooltip-pad-info-monster-extended" ' \
      "tigcode=\"#{monster_id}\" " \
      'href="http://www.thisisgame.com/pad/info/monster/detail.php' \
      "?code=#{monster_id}\">" \
      "<img src=\"#{monster_icon_link(monster_id, grayscale: grayscale)}\"></a>"
  end
end

def monster_icon(monster_id, grayscale: false, combined: false)
  if combined
    icon_file = monster_id.map { |id| MAP.fetch(id, id) }.join('_')
    "icons/combined/#{icon_file}.png"
  else
    icon_file = MAP.fetch(monster_id, monster_id)
    "icons/#{'grayscale/' if grayscale}#{icon_file}.png"
  end
end

def monster_icon_link(monster_id, grayscale: false, combined: false)
  'https://raw.githubusercontent.com/yous/pad_monster_book/master/assets/' +
    monster_icon(monster_id, grayscale: grayscale, combined: combined)
end

def process(item)
  case item
  when nil
    ''
  when String
    item
  when Array
    item.map { |x| process(x) }.join
  when Hash
    process_hash(item)
  end
end

def process_hash(item)
  case item.keys.first
  when 'box' then %(<h4 class="pad-title7 w430 cyan1">#{item['box']}</h4>)
  when 'link' then %(<a href="#{item['link']}">#{item['link']}</a>)
  when 'anchor' then %(<a href="##{item['anchor']}">#{item['anchor']}</a>)
  when 'sort'
    %(<a name="#{item['sort']}">&nbsp;</a>) +
      %(<h4 class="pad-title7 w430 cyan1">#{item['sort']}</h4><br>) +
      process_sort(SORT[item['sort']])
  end
end

def process_sort(sort)
  sort.map { |monster_row| process_sort_row(monster_row) + '<br>' }.join
end

def process_sort_row(row)
  return '' unless row
  if row['grayscale']
    row['items'].map { |id| monster_link(id, grayscale: true) }.join
  elsif row['combined']
    row['items'].map { |ids| monster_link(ids, combined: true) }.join
  else
    row['items'].map { |monster| process_sort_item(monster) }.join
  end
end

def process_sort_item(monster)
  return monster_link(monster) if monster.is_a?(Fixnum)
  if monster['grayscale']
    monster_link(monster['item'], grayscale: true)
  elsif monster['combined']
    monster_link(monster['item'], combined: true)
  end
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
  # Get the list of grayscale monster.
  grayscale_monster = []
  SORT.values.flatten(1).each do |monster_row|
    # -
    next unless monster_row
    # - grayscale: true
    #   items: [122, 124, 126, 128, 130]
    if monster_row['grayscale']
      monster_row['items'].each do |monster_id|
        next if File.exist?(
          asset_file(monster_icon(monster_id, grayscale: true)))
        grayscale_monster << monster_id
      end
    # - items:
    #   - grayscale: true
    #     item: 648
    elsif !monster_row['combined']
      monster_row['items'].select { |x| x.is_a?(Hash) && x['grayscale'] }
        .each do |monster|
        next if File.exist?(
          asset_file(monster_icon(monster['item'], grayscale: true)))
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
    image = Magick::ImageList.new(asset_file(monster_icon(monster_id)))
    grayscale_image = image.cur_image.quantize(256, Magick::GRAYColorspace,
                                               Magick::NoDitherMethod)
    grayscale_image.write(asset_file(monster_icon(monster_id, grayscale: true)))
  end
  puts 'Done.'
end

desc 'Generate combined icons'
task :combine do
  # Get the list of combined icon.
  combined_icons = []
  SORT.values.flatten(1).each do |monster_row|
    # -
    next unless monster_row
    # - combined: true
    #   items:
    #   - [388, 389]
    #   - [390, 391]
    if monster_row['combined']
      monster_row['items'].each do |combined|
        next if File.exist?(
          asset_file(monster_icon(combined, combined: true)))
        combined_icons << combined
      end
    # - items:
    #   - combined: true
    #     item: [755, 756]
    elsif !monster_row['grayscale']
      monster_row['items'].select { |x| x.is_a?(Hash) && x['combined'] }
        .each do |monster|
        next if File.exist?(
          asset_file(monster_icon(monster['item'], combined: true)))
        combined_icons << monster['item']
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
    above_image = Magick::ImageList.new(asset_file(monster_icon(above_id)))
    below_image = Magick::ImageList.new(asset_file(monster_icon(below_id)))

    unless above_image.columns == below_image.columns &&
      above_image.rows == below_image.rows
      warn 'Images have different size: ' \
        "#{above_id}: #{above_image.columns}x#{above_image.rows}, " \
        "#{below_id}: #{below_image.columns}x#{below_image.rows}"
      next
    end

    above_image.rows.times do |y|
      above_image.columns.times.select { |x| x <= y }.each do |x|
        above_image.pixel_color(x, y, below_image.pixel_color(x, y))
      end
    end
    above_image.write(asset_file(monster_icon(monster_ids, combined: true)))
  end
  puts 'Done.'
end

desc 'Generate article based on article.yml'
task :generate do
  puts 'Generating article...'
  generated_article = ''
  ARTICLE.each do |line|
    generated_article << process(line)
    generated_article << '<br>'
  end

  article_html = File.read(asset_file('article.html'))
  if generated_article == article_html
    puts 'Already up-to-date.'
    exit
  end
  File.open(asset_file('article.html'), 'wb') { |f| f.write(generated_article) }
  puts 'Done.'
end

require 'rubocop/rake_task'
RuboCop::RakeTask.new(:rubocop)

task default: :rubocop
