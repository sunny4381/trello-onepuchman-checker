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

  def find_latest_topic
    latest_topic = nil
    100.upto(200).each do |topic|
      break unless exists_topic?(topic)
      latest_topic = topic
    end
    latest_topic
  end

  def configure_trello
    Trello.configure do |config|
      config.consumer_key = ENV['TRELLO_CONSUMER_KEY']
      config.consumer_secret = ENV['TRELLO_CONSUMER_SECRET']
      config.oauth_token = ENV['TRELLO_OAUTH_TOKEN']
    end
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

  def read_latest_topic
    list = trello_list
    card = list.cards.find do |card|
      card.name == '.topic'
    end
    unless card
      card = Trello::Card.create(
        name: '.topic',
        list_id: list.id,
        desc: 'このカードは消さないでください。',
        pos: 'top')
    end

    comment = card.actions.find do |action|
      action.type == 'commentCard'
    end
    return nil unless comment

    topic = comment.data['text'].to_i
    return nil if topic == 0
    topic
  end

  def write_latest_topic(topic)
    list = trello_list
    card = list.cards.find do |card|
      card.name == '.topic'
    end
    unless card
      card = Trello::Card.create(
        name: '.topic',
        list_id: list.id,
        desc: 'このカードは消さないでください。',
        pos: 'top')
    end

    comment = card.actions.find do |action|
      action.type == 'commentCard'
    end

    if comment
      # there is no convenient method to update comment
      list.client.put("/cards/#{card.id}/actions/#{comment.id}/comments", text: topic.to_s)
    else
      card.add_comment(topic.to_s)
    end
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

  task :dotenv do
    Dotenv.load
  end

  task(configure_trello: [:dotenv]) do
    configure_trello
  end

  task(init: [:dotenv, :configure_trello]) do
    unless read_latest_topic
      write_latest_topic(find_latest_topic)
    end
  end

  desc '新しい話数が投稿されたか調べる'
  task(check: [:dotenv, :configure_trello, :init]) do
    topic = read_latest_topic

    if exists_topic?(topic + 1)
      send_update_info(topic + 1)
      write_latest_topic(topic + 1)
    end
  end

  task(test: [:dotenv, :configure_trello]) do
    list = trello_list
    puts "#{list.board.name} - #{list.name}"
    card = list.cards.find do |card|
      card.name == '.topic'
    end
    unless card
      card = Trello::Card.create(
        name: '.topic',
        list_id: list.id,
        desc: 'このカードは消さないでください。',
        pos: 'top')
    end

    comment = card.actions.find do |action|
      action.type == 'commentCard'
    end

    puts comment.data['text']
    list.client.put("/cards/#{card.id}/actions/#{comment.id}/comments", text: '103')
  end
end

# task(default: 'onepunchman:check')
