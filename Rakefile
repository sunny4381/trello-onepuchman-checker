require 'dotenv'
require 'faraday'
require 'logger'
require 'trello'

namespace :onepunchman do
  def logger
    @logger ||= Logger.new(STDOUT)
  end

  def conn
    @conn ||= Faraday.new(url: 'http://galaxyheavyblow.web.fc2.com/') do |builder|
      builder.request(:url_encoded)
      builder.use(Faraday::Response::Logger, logger)
      builder.adapter(Faraday.default_adapter)
    end
  end

  def exists_topic?(topic)
    response = conn.get("/%{topic}.html" % { topic: topic })
    response.status == 200
  end

  def latest_topic_file
    @latest_topic_file ||= begin
      file = File.expand_path('tmp/latest_topic.txt', File.dirname(__FILE__))
      dir = File.dirname(file)
      Dir.mkdir(dir) unless Dir.exists?(dir)
      file
    end
  end

  def read_latest_topic
    File.read(latest_topic_file).strip.to_i
  end

  def write_latest_topic(topic)
    open(latest_topic_file, 'w') do |f|
      f.puts(topic)
    end
    nil
  end

  def send_update_info(topic)
    url = "http://galaxyheavyblow.web.fc2.com/%{topic}.html" % { topic: topic }
    list = trello_list
    card = Trello::Card.create(
      name: "ワンパンマン 第#{topic}話",
      list_id: list.id,
      desc: "新しい話題 第#{topic}話があります。\n#{url}",
      pos: 'bottom')
  end

  def trello_list
    @list ||= begin
      board = Trello::Board.all.find do |board|
        board.name == ENV['TRELLO_BOARD_NAME']
      end
      return nil unless board
      list = board.lists.find do |list|
        list.name == ENV['TRELLO_LIST_NAME']
      end
      list
    end
  end

  task :dotenv do
    Dotenv.load
  end

  task(trello: [:dotenv]) do
    Trello.configure do |config|
      config.consumer_key = ENV['TRELLO_CONSUMER_KEY']
      config.consumer_secret = ENV['TRELLO_CONSUMER_SECRET']
      config.oauth_token = ENV['TRELLO_OAUTH_TOKEN']
    end
  end

  task(init: [:dotenv]) do
    unless File.exists?(latest_topic_file)
      100.upto(200).each do |topic|
        break unless exists_topic?(topic)
        write_latest_topic(topic)
      end
    end
  end

  desc '新しい話数が投稿されたか調べる'
  task(check: [:dotenv, :init, :trello]) do
    topic = read_latest_topic

    if exists_topic?(topic + 1)
      send_update_info(topic + 1)
      write_latest_topic(topic + 1)
    end
  end
end

# task(default: 'onepunchman:check')
