class Meetup
  POSTS = File.dirname(__FILE__) + '/_posts'

  attr_accessor :date
  attr_accessor :pub

  def self.exists? pub
    m = Meetup.new
    m.pub = pub
    Dir[POSTS + "/*-#{m.slug}.md"].any?
  end

  def self.next
    m = Meetup.new
    m.date = (Dir[POSTS + '/*.md'].map { |file_name|
      stamp = File.basename(file_name, '.md').scan(/(\d+-\d+-\d+)-.*/)[0][0]
      Time.parse stamp
    }.sort[-1] || Time.now).to_date.next_month
    m
  end

  def slug
    pub.name.downcase.strip.gsub(/\s+/, '-').gsub(/[^a-z0-9\-]/, '')
  end

  def template
    data = File.read "_templates/meetup.md.erb"
    ERB.new data
  end

  def file_name
    date.strftime("%Y-%m-%d") + '-' + slug + '.md'
  end

  def save
    File.open [POSTS, file_name].join('/'), 'w+' do |f|
      f.puts template.result send(:binding)
    end
  end
end

task :choose_next_pub do
  Bundler.require
  require 'date'
  require 'erb'

  s = BeerInTheEvening::Search.new
  s.tube_station = [
    BeerInTheEvening::Location::Tube::HOLBORN,
    BeerInTheEvening::Location::Tube::OXFORD_CIRCUS,
    BeerInTheEvening::Location::Tube::PICCADILLY_CIRCUS,
    BeerInTheEvening::Location::Tube::GOODGE_STREET
  ].sort_by{rand}[0]
  s.wifi = true
  s.food = true
  s.real_ale = true
  s.minimum_rating = 6.5

  puts "There are #{s.number_of_results} possible results with these parameters"

  candidates = []
  s.each do |pub|
    next unless !pub.visited?
    puts "Candidate: #{pub}"
    candidates << pub
    break if candidates.size >= 5
  end
  puts "Got #{candidates.size} candidates"

  if candidates.empty?
    puts "Couldn't find any candidate pubs"
    exit 1
  end
  selected_pub = candidates.sort_by{rand}[0]
  puts "Selected #{selected_pub.name}: #{selected_pub.url}"
  meetup = Meetup.next
  meetup.pub = selected_pub
  meetup.save
end
