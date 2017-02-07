---
layout: post
title:  "Seven Languages in Seven Weeks"
date:   2017-02-07 15:23:00 +0800
categories: programming ruby technical
---
This morning, after 10 minutes struggle of do or do not follow the self study section.  Just spend several hours of searching, coding and debugging, finally I got the following ruby script that is able to pass the self study excise.

``` diff
diff -r 9efeca9bd507 learning/ruby/acts_as_csv_module.rb
--- a/learning/ruby/acts_as_csv_module.rb	Tue Feb 07 15:13:11 2017 +0800
+++ b/learning/ruby/acts_as_csv_module.rb	Tue Feb 07 15:13:49 2017 +0800
@@ -1,32 +1,90 @@
 module ActsAsCsv
+
   def self.included(base)
     base.extend ClassMethods
   end
+
   module ClassMethods
+    
     def acts_as_csv
       include InstanceMethods
     end
+
+    def acts_as_enumerable
+      include Enumerable
+    end
   end
+
   module InstanceMethods
+    
     def read
       @csv_contents = []
       filename = self.class.to_s.downcase + '.txt'
       file = File.new(filename)
-      @headers = file.gets.chomp.split(', ' )
+      CsvRow.headers = file.gets.chomp.split(', ' )
+
+      # puts "CsvRow.headers is #{CsvRow.headers}"
       file.each do |row|
-        @csv_contents << row.chomp.split(', ' )
+        csvrow = CsvRow.new
+        csvrow.row_content = row.chomp.split(', ' )
+
+        # puts "csvrow is #{csvrow.row_content}"
+	@csv_contents << csvrow
       end
     end
-    attr_accessor :headers, :csv_contents
+    
+    attr_accessor :csv_contents
+    
     def initialize
       read
     end
+
+    def each(&block)
+      # puts "csv_contents.each is #{@csv_contents.each}"
+      @csv_contents.each(&block)
+    end
+    
   end
 end
+
 class RubyCsv # no inheritance! You can mix it in
   include ActsAsCsv
   acts_as_csv
+  acts_as_enumerable
 end
-m = RubyCsv.new
-puts m.headers.inspect
-puts m.csv_contents.inspect
+
+class CsvRow
+  @@header_indexes = {}
+  
+  def method_missing name, *args
+    @row_content[@@header_indexes[name.to_s]]
+  end
+
+  attr_accessor :row_content
+
+  def initialize
+    @row_content = []
+  end
+
+  def self.headers=(h)
+    @@headers = h
+
+    # puts "h is #{h}"
+    h.each_index {|i| @@header_indexes[h[i]] = i}
+    # puts "@@header_indexes is #{@@header_indexes}"
+  end
+
+  def self.headers
+    @@headers
+  end
+end
+
+
+# m = RubyCsv.new
+# puts m.headers.inspect
+# puts m.csv_contents.inspect
+
+csv = RubyCsv.new
+
+# puts "csv_contents is #{csv.csv_contents}"
+csv.each {|row| puts row.one}
```

# Ruby feature learned

1. Class variable is defined and accessed with "double at" prefix `@@`.
2. Class accessor must be defined with `self.` prefix, otherwise the result method will be instance accessor, I was confused by the complex syntax sugar used in the [link][classaccessor].
3. Array has convenient `.each_index` method that will save you lots type.
4. [`each`][eachforobject] method have to be defiend as `each(&block)` to be able to work, and the inside `each` being invoked should also be invoked as `.each(&block)` for the block to be passed into it.
5. [`attr_accessor`][tivarExpecting] will generate ruby specific getter and setters codes for decorated attributes(have to be specified in the form of [`:rubysymbol`][colonruby]).

[colonruby]: http://stackoverflow.com/questions/6337897/what-is-the-colon-operator-in-ruby
[tivarExpecting]: http://stackoverflow.com/questions/21625992/syntax-error-unexpected-tivar-expecting
[eachforobject]: https://stackoverflow.com/questions/2080007/how-do-i-add-each-method-to-ruby-object-or-should-i-extend-array/2080045#2080045
[classaccessor]: http://stackoverflow.com/questions/4202878/accessing-a-class-instance-variable-from-outside
