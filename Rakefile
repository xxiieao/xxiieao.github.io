desc "create new post"
task :post, [:filename, :categores] do |t, args|
  path = File.dirname(__FILE__) + "/_posts/#{Time.now.strftime('%Y-%m-%d')}-#{args[:filename]}.md"
  new_file = File.new(path, "w")
  new_file << ["---",
  	           "layout: post",
  	           "title:  \"#{args[:filename].gsub("-", "")}\"",
               "date: #{Time.now.strftime('%Y-%m-%d %H:%m:%S')}",
               "categories: \"#{args[:categores]}\"",
               "author: xxiieao",
               "---"].join("\n")
  new_file.close
end