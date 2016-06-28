class GroupScraper
  def initialize(access_token, group_id)
    @access_token = access_token
    @group_id     = group_id
    @url          =  "https://graph.facebook.com/126553607367124/feed?access_token=#{@access_token}"
    @data         = []
  end

  def start
    scrape(@url)
  end

  def scrape(url)
    resp = Crack::JSON.parse(RestClient.get(url))

    if resp['data'] && resp['data'].length > 0
      resp['data'].each do |fb_post|
        post = {
          :fb_id           => fb_post['id'],
          :fb_author       => fb_post["from"]["name"],
          :fb_author_id    => fb_post["from"]["id"],
          :message         => fb_post["message"],
          :fb_created_time => fb_post["created_time"],
          :fb_updated_time => fb_post["updated_time"]
        }
        p post
        @data << post
        if fb_post['comments'] && fb_post['comments']['data']
          fb_post['comments']['data'].each do |fb_comment|
            comment = {
              :fb_id           => fb_comment['id'],
              :fb_author       => fb_comment["from"]["name"],
              :fb_author_id    => fb_comment["from"]["id"],
              :message         => fb_comment["message"],
              :fb_created_time => fb_comment["created_time"],
              :fb_likes        => fb_comment['likes']
            }
            p comment
            @data << comment
          end
        end
      end
      
      if resp['paging']['next']
        scrape(resp['paging']['next'])
      end      
    else
      return
    end
  end

  def to_csv
    FasterCSV.open("fb_posts_126553607367124.csv", "w") do |csv|
      csv << %w[name fb_id date text url]
      @data.each do |post|
        csv << [post[:fb_author], post[:fb_id], post[:fb_created_time], post[:message], "https://www.facebook.com/groups/126553607367124{post[:fb_id].split(/_/)[0]}/permalink/#{post[:fb_id].split(/_/)[1]}"]
      end
    end
  end
end

if __FILE__ == $0
  gs = GroupScraper.new(ARGV[0], ARGV[1])
  gs.start
  gs.to_csv
end