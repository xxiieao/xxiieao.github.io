desc "create new post"
task :post, [:filename, :categores, :datetime] do |t, args|
  date = args[:datetime] || Time.now.strftime('%Y-%m-%d %H:%m:%S +1200')
  path = File.dirname(__FILE__) + "/_posts/#{date[0..9]}-#{args[:filename].gsub(" ","-")}.md"
  new_file = File.new(path, "w")
  new_file << ["---",
  	           "layout: post",
  	           "title:  \"#{args[:filename]}\"",
               "date: #{date}",
               "categories: \"#{args[:categores]}\"",
               "author: xxiieao",
               "---"].join("\n")
  new_file.close
end
