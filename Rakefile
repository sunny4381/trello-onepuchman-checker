require 'dotenv'
require 'faraday'
require 'logger'
require 'trello'
require 'uri'

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

  def find_latest_topic
    response = conn.get
    return nil if response.status != 200

    # convert encoding
    body = response.body.dup
    body.force_encoding('Shift_JIS')
    body.encode!('UTF-8')

    # extract topic links
    links = body.scan(/<a href=".+?">.+?<\/a>/).map do |link|
      link =~ /<a href="(.+?)">(.+?)<\/a>/
      { href: $1, text: $2 }
    end
    links.each do |link|
      link[:href] = "http://galaxyheavyblow.web.fc2.com/#{link[:href]}" if !link[:href].start_with?('http')
    end
    links.each do |link|
      link[:text] = link[:text].tr('０-９', '0-9')
    end
    links.each do |link|
      if /^第(\d+(\.\d+)?)話$/ =~ link[:text]
        topic = $1.to_f
        topic = topic.to_i if topic == topic.to_i
        link[:topic] = topic
      else
        link[:topic] = nil
      end
    end
    links.reject! do |link|
      link[:topic].nil?
    end

    # find latest topic link
    links.max_by { |link| link[:topic] }
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

    topic = begin
      JSON.parse(comment.data['text']).symbolize_keys
    rescue JSON::ParserError
      topic = comment.data['text'].to_i
      {
        href: "http://galaxyheavyblow.web.fc2.com/#{topic}",
        text: "第#{topic}話",
        topic: topic
      }.symbolize_keys
    end
    topic
  end

  def write_latest_topic(link)
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
      list.client.put("/cards/#{card.id}/actions/#{comment.id}/comments", text: link.to_json)
    else
      card.add_comment(link.to_json)
    end
  end

  def send_update_info(link)
    list = trello_list
    card = Trello::Card.create(
      name: "ワンパンマン 第#{link[:topic]}話",
      list_id: list.id,
      desc: "新しい話題 第#{link[:topic]}話があります。\n#{link[:href]}",
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
    latest_topic = find_latest_topic

    if topic[:topic] < latest_topic[:topic]
      send_update_info(latest_topic)
      write_latest_topic(latest_topic)
    end
  end

  # task(test: [:dotenv, :configure_trello]) do
  #   puts read_latest_topic
  # end
end

# task(default: 'onepunchman:check')
