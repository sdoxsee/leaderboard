# leaderboard

Leaderboards backed by Redis in Ruby, http://redis.io.

Builds off ideas proposed in http://blog.agoragames.com/2011/01/01/creating-high-score-tables-leaderboards-using-redis/.

## Installation

`gem install leaderboard`

or in your `Gemfile`

```ruby
gem 'leaderboard'
```

Make sure your redis server is running! Redis configuration is outside the scope of this README, but 
check out the Redis documentation, http://redis.io/documentation.

## Compatibility

The gem has been built and tested under Ruby 1.8.7, Ruby 1.9.2 and Ruby 1.9.3
	
## Usage

Create a new leaderboard or attach to an existing leaderboard named 'highscores':

```ruby
  ruby-1.9.2-p180 :002 > highscore_lb = Leaderboard.new('highscores')
   => #<Leaderboard:0x0000010307b530 @leaderboard_name="highscores", @page_size=25, @redis_connection=#<Redis client v2.2.2 connected to redis://localhost:6379/0 (Redis v2.2.5)>>
```
    
If you need to pass in options for Redis, you can do this in the initializer:

```ruby
  ruby-1.9.2-p180 :007 > redis_options = {:host => 'localhost', :port => 6379, :db => 1}
   => {:host=>"localhost", :port=>6379, :db=>1} 
  ruby-1.9.2-p180 :008 > highscore_lb = Leaderboard.new('highscores', Leaderboard::DEFAULT_OPTIONS, redis_options)
   => #<Leaderboard:0x00000103095200 @leaderboard_name="highscores", @page_size=25, @redis_connection=#<Redis client v2.2.2 connected to redis://localhost:6379/1 (Redis v2.2.5)>> 
```

The `Leaderboard::DEFAULT_OPTIONS` are as follows:

```ruby
DEFAULT_OPTIONS = {
  :page_size => DEFAULT_PAGE_SIZE,
  :reverse => false
}
```

You would use the option, `:reverse => true`, if you wanted a leaderboard sorted from lowest to highest score.

You can pass in an existing connection to Redis using :redis_connection in the Redis options hash:

```ruby
  ruby-1.9.2-p180 :009 > redis = Redis.new
   => #<Redis client v2.2.2 connected to redis://127.0.0.1:6379/0 (Redis v2.2.5)> 
  ruby-1.9.2-p180 :010 > redis_options = {:redis_connection => redis}
   => {:redis_connection=>#<Redis client v2.2.2 connected to redis://127.0.0.1:6379/0 (Redis v2.2.5)>} 
  ruby-1.9.2-p180 :011 > highscore_lb = Leaderboard.new('highscores', Leaderboard::DEFAULT_OPTIONS, redis_options)
   => #<Leaderboard:0x000001028791e8 @leaderboard_name="highscores", @page_size=25, @redis_connection=#<Redis client v2.2.2 connected to redis://127.0.0.1:6379/0 (Redis v2.2.5)>> 
```
 
You can set the page size to something other than the default page size (25):

```ruby
  ruby-1.9.2-p180 :012 > highscore_lb.page_size = 5
   => 5 
  ruby-1.9.2-p180 :013 > highscore_lb
   => #<Leaderboard:0x000001028791e8 @leaderboard_name="highscores", @page_size=5, @redis_connection=#<Redis client v2.2.2 connected to redis://127.0.0.1:6379/0 (Redis v2.2.5)>> 
```
  
Add members to your leaderboard using rank_member:

```ruby
  ruby-1.9.2-p180 :014 > 1.upto(10) do |index|
  ruby-1.9.2-p180 :015 >     highscore_lb.rank_member("member_#{index}", index)
  ruby-1.9.2-p180 :016?>   end
   => 1 
```

You can call rank_member with the same member and the leaderboard will be updated automatically.

Get some information about your leaderboard:

```ruby
  ruby-1.9.2-p180 :020 > highscore_lb.total_members
   => 10 
  ruby-1.9.2-p180 :021 > highscore_lb.total_pages
   => 1 
```
  
Get some information about a specific member(s) in the leaderboard:

```ruby
  ruby-1.9.2-p180 :022 > highscore_lb.score_for('member_4')
   => 4.0 
  ruby-1.9.2-p180 :023 > highscore_lb.rank_for('member_4')
   => 7 
  ruby-1.9.2-p180 :024 > highscore_lb.rank_for('member_10')
   => 1 
```
  
Get page 1 in the leaderboard:

```ruby
  ruby-1.9.2-p180 :025 > highscore_lb.leaders(1)
   => [{:member=>"member_10", :rank=>1, :score=>10.0}, {:member=>"member_9", :rank=>2, :score=>9.0}, {:member=>"member_8", :rank=>3, :score=>8.0}, {:member=>"member_7", :rank=>4, :score=>7.0}, {:member=>"member_6", :rank=>5, :score=>6.0}, {:member=>"member_5", :rank=>6, :score=>5.0}, {:member=>"member_4", :rank=>7, :score=>4.0}, {:member=>"member_3", :rank=>8, :score=>3.0}, {:member=>"member_2", :rank=>9, :score=>2.0}, {:member=>"member_1", :rank=>10, :score=>1.0}] 
```
	
You can pass various options to the calls `leaders`, `around_me` and `ranked_in_list`. Valid options are `:with_scores`, `:with_rank`, `:use_zero_index_for_rank` and `:page_size`.
Below is an example of retrieving the first page in the leaderboard without ranks:

```ruby
  ruby-1.9.2-p180 :026 > highscore_lb.leaders(1, :with_rank => false)
   => [{:member=>"member_10", :score=>9.0}, {:member=>"member_9", :score=>7.0}, {:member=>"member_8", :score=>5.0}, {:member=>"member_7", :score=>3.0}, {:member=>"member_6", :score=>1.0}, {:member=>"member_5", :score=>0.0}, {:member=>"member_4", :score=>0.0}, {:member=>"member_3", :score=>0.0}, {:member=>"member_2", :score=>0.0}, {:member=>"member_1", :score=>0.0}] 
```

Below is an example of retrieving the first page in the leaderboard without scores or ranks:

```ruby
  ruby-1.9.2-p180 :028 > highscore_lb.leaders(1, :with_scores => false, :with_rank => false)
   => [{:member=>"member_10"}, {:member=>"member_9"}, {:member=>"member_8"}, {:member=>"member_7"}, {:member=>"member_6"}, {:member=>"member_5"}, {:member=>"member_4"}, {:member=>"member_3"}, {:member=>"member_2"}, {:member=>"member_1"}] 
```

Add more members to your leaderboard:

```ruby
  ruby-1.9.2-p180 :029 > 50.upto(95) do |index|
  ruby-1.9.2-p180 :030 >     highscore_lb.rank_member("member_#{index}", index)
  ruby-1.9.2-p180 :031?>   end
   => 50 
  ruby-1.9.2-p180 :032 > highscore_lb.total_pages
   => 3 
```
  
Get an "Around Me" leaderboard for a member:

```ruby
  ruby-1.9.2-p180 :033 > highscore_lb.around_me('member_53')
   => [{:member=>"member_65", :rank=>31, :score=>65.0}, {:member=>"member_64", :rank=>32, :score=>64.0}, {:member=>"member_63", :rank=>33, :score=>63.0}, {:member=>"member_62", :rank=>34, :score=>62.0}, {:member=>"member_61", :rank=>35, :score=>61.0}, {:member=>"member_60", :rank=>36, :score=>60.0}, {:member=>"member_59", :rank=>37, :score=>59.0}, {:member=>"member_58", :rank=>38, :score=>58.0}, {:member=>"member_57", :rank=>39, :score=>57.0}, {:member=>"member_56", :rank=>40, :score=>56.0}, {:member=>"member_55", :rank=>41, :score=>55.0}, {:member=>"member_54", :rank=>42, :score=>54.0}, {:member=>"member_53", :rank=>43, :score=>53.0}, {:member=>"member_52", :rank=>44, :score=>52.0}, {:member=>"member_51", :rank=>45, :score=>51.0}, {:member=>"member_50", :rank=>46, :score=>50.0}, {:member=>"member_10", :rank=>47, :score=>10.0}, {:member=>"member_9", :rank=>48, :score=>9.0}, {:member=>"member_8", :rank=>49, :score=>8.0}, {:member=>"member_7", :rank=>50, :score=>7.0}, {:member=>"member_6", :rank=>51, :score=>6.0}, {:member=>"member_5", :rank=>52, :score=>5.0}, {:member=>"member_4", :rank=>53, :score=>4.0}, {:member=>"member_3", :rank=>54, :score=>3.0}, {:member=>"member_2", :rank=>55, :score=>2.0}] 
```
	
Get rank and score for an arbitrary list of members (e.g. friends):

```ruby
  ruby-1.9.2-p180 :034 > highscore_lb.ranked_in_list(['member_1', 'member_62', 'member_67'])
   => [{:member=>"member_1", :rank=>56, :score=>1.0}, {:member=>"member_62", :rank=>34, :score=>62.0}, {:member=>"member_67", :rank=>29, :score=>67.0}]
```   

### Other useful methods

```
  delete_leaderboard: Delete the current leaderboard  
  remove_member(member): Remove a member from the leaderboard
  total_members: Total # of members in the leaderboard
  total_pages: Total # of pages in the leaderboard given the leaderboard's page_size	
  total_members_in_score_range(min_score, max_score): Count the number of members within a score range in the leaderboard
  change_score_for(member, delta): Change the score for a member by some amount delta (delta could be positive or negative)
  rank_for(member): Retrieve the rank for a given member in the leaderboard
  score_for(member): Retrieve the score for a given member in the leaderboard
  check_member?(member): Check to see whether member is in the leaderboard
  score_and_rank_for(member): Retrieve the score and rank for a member in a single call
  remove_members_in_score_range(min_score, max_score): Remove members from the leaderboard within a score range
  percentile_for(member): Calculate the percentile for a given member
  merge_leaderboards(destination, keys, options = {:aggregate => :min}): Merge leaderboards given by keys with this leaderboard into destination
  intersect_leaderboards(destination, keys, options = {:aggregate => :min}): Intersect leaderboards given by keys with this leaderboard into destination
```

Check the [online documentation](http://rubydoc.info/github/agoragames/leaderboard/master/frames) for more detail on each method.
      
## Performance Metrics

10 million sequential scores insert:

```ruby
  ruby-1.9.2-p180 :003 > highscore_lb = Leaderboard.new('highscores')
   => #<Leaderboard:0x0000010205fc50 @leaderboard_name="highscores", @page_size=25, @redis_connection=#<Redis client v2.2.2 connected to redis://localhost:6379/0 (Redis v2.2.5)>> 
  ruby-1.9.2-p180 :004 > insert_time = Benchmark.measure do
  ruby-1.9.2-p180 :005 >     1.upto(10000000) do |index|
  ruby-1.9.2-p180 :006 >       highscore_lb.rank_member("member_#{index}", index)
  ruby-1.9.2-p180 :007?>     end
  ruby-1.9.2-p180 :008?>   end
   => 323.070000 148.560000 471.630000 (942.068307)
```
  
Average time to request an arbitrary page from the leaderboard:

```ruby
  ruby-1.9.2-p180 :009 > requests_to_make = 50000
   => 50000 
  ruby-1.9.2-p180 :010 > lb_request_time = 0
   => 0 
  ruby-1.9.2-p180 :011 > 1.upto(requests_to_make) do
  ruby-1.9.2-p180 :012 >     lb_request_time += Benchmark.measure do
  ruby-1.9.2-p180 :013 >       highscore_lb.leaders(rand(highscore_lb.total_pages))
  ruby-1.9.2-p180 :014?>     end.total
  ruby-1.9.2-p180 :015?>   end
   => 1 
  ruby-1.9.2-p180 :016 > p lb_request_time / requests_to_make
  0.001513999999999998
   => 0.001513999999999998 
```
     
10 million random scores insert:

```ruby
  ruby-1.9.2-p180 :018 > insert_time = Benchmark.measure do
  ruby-1.9.2-p180 :019 >     1.upto(10000000) do |index|
  ruby-1.9.2-p180 :020 >       highscore_lb.rank_member("member_#{index}", rand(50000000))
  ruby-1.9.2-p180 :021?>     end
  ruby-1.9.2-p180 :022?>   end
   => 338.480000 155.200000 493.680000 (2188.702475)
```
  
Average time to request an arbitrary page from the leaderboard:

```ruby
  ruby-1.9.2-p180 :007 > 1.upto(requests_to_make) do
  ruby-1.9.2-p180 :008 >     lb_request_time += Benchmark.measure do
  ruby-1.9.2-p180 :009 >       highscore_lb.leaders(rand(highscore_lb.total_pages))
  ruby-1.9.2-p180 :010?>     end.total
  ruby-1.9.2-p180 :011?>   end
   => 1 
  ruby-1.9.2-p180 :012 > p lb_request_time / requests_to_make
  0.0014615999999999531
   => 0.0014615999999999531 
```

## Future Ideas

* Bulk insert

## Ports

The following ports have been made of the leaderboard gem.

* Java: https://github.com/agoragames/java-leaderboard
* NodeJS: https://github.com/omork/node-leaderboard
* PHP: https://github.com/agoragames/php-leaderboard
* Python: https://github.com/agoragames/python-leaderboard
* Scala: https://github.com/agoragames/scala-leaderboard
  
## Contributing to leaderboard
 
* Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet
* Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it
* Fork the project
* Start a feature/bugfix branch
* Commit and push until you are happy with your contribution
* Make sure to add tests for it. This is important so I don't break it in a future version unintentionally.
* Please try not to mess with the Rakefile, version, or history. If you want to have your own version, or is otherwise necessary, that is fine, but please isolate to its own commit so I can cherry-pick around it.

## Copyright

Copyright (c) 2011-2012 David Czarnecki. See LICENSE.txt for further details.
