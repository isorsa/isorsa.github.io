---
title: Recursively count all nodes in the polymorphic adjacency tree
date: 2022-02-01 12:13:14
---

<p>There are several ways to count nodes in the adjacency tree (parent-child model). In our example, polymorphic means that parent node could be a story or a comment. A story could have many comments and a comment can have many comments.</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/number_of_comments.png" alt="number of comments" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/comments_tree.png" alt="comments tree" />
  </div>
</div>

<p>One of the ways is to recursively count nodes in the code.</p>
<p>Here is an example:</p>

<pre>
  def slow_count_comments(story)
    count = story.comments.count
    story.comments.each { |comment| count += count_comments(comment) }
    count
  end
</pre>

<p>The problem with this approach is, that it is slow, especially, when we need to count nodes in a deeply nested tree.</p>
<p>The <code>slow_count_comments</code> method makes over 300 sql queries which takes ~450ms with rendering. Here is the output from the rack profiler:</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/slow_query.png" alt="slow profiler" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/slow_query_details.png" alt="slow profiler" />
  </div>
</div>

<p>Another way is to count nodes with a recursive query. In PostgreSQL 8.4+ it can be done with Common Table Expressions (CTE). Here is an example of recursive CTE:</p>

<pre>
  def fast_count_comments(story)
    sql = <<-SQL
      WITH RECURSIVE story_comments AS (
        SELECT id, commentable_id FROM comments WHERE commentable_id = #{story.id} AND commentable_type = 'Story'
        UNION
        SELECT c.id, c.commentable_id FROM comments AS c
        INNER JOIN story_comments AS sc ON ((sc.id = c.commentable_id) AND (c.commentable_type = 'Comment'))
      ) SELECT count(*) FROM story_comments;
    SQL
    count = Comment.count_by_sql(sql)
    count
  end
</pre >

<p>The CTE expression consists of 5 steps:</p>
<ol>
  <li>Non-recursive subquery:
    <pre>SELECT id, commentable_id FROM comments WHERE commentable_id = #{story.id} AND commentable_type = 'Story'</pre>
  </li>
  <li>Recursive subquery:
    <pre>SELECT c.id, c.commentable_id FROM comments AS c <br />INNER JOIN story_comments AS sc ON ((sc.id = c.commentable_id) AND (c.commentable_type = 'Comment'))</pre>
  </li>
  <li>Recursively repeat step 2</li>
  <li>Combine results with UNION</li>
  <li>Count results with <pre>SELECT count(*) FROM story_comments</pre></li>
</ol>

<p>Recursive CTE implementaion is much faster. The <code>fast_count_comments</code> method makes 1 sql query (~20ms with rendering). Here is the output from the rack profiler:</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/fast_query.png" alt="fast profiler" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/fast_query_details.png" alt="fast profiler" />
  </div>
</div>

<p>Here is a<a href="https://github.com/isorsa/threaded-comments"> threaded comments application.</a></p>
