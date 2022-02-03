---
title: Recursively count all comments in the adjacency tree
date: 2022-02-01 12:13:14
---

<p>There are several ways to count nodes in the adjacency tree (parent-child model).</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/number_of_comments.png" alt="number of comments" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/comments_tree.png" alt="comments tree" />
  </div>
</div>

<p>One of the ways is to count nodes recursively in the code. Each story has 155 comments.</p>
<p>Here is a slow counter:</p>

<pre>
  def slow_count_comments(story)
    count = story.comments.count
    story.comments.each { |comment| count += count_comments(comment) }
    count
  end
</pre>

<p>There is a problem with this approach though. It is very slow, when we deal with a deeply nested tree.</p>
<p>The <code>slow_count_comments</code> method makes over 300 sql queries (~450ms with rendering). Here is the output from the rack mini profiler:</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/slow_query.png" alt="slow profiler" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/slow_query_details.png" alt="slow profiler" />
  </div>
</div>

<p>Another way is to count nodes with a recursive query. In PostgreSQL 8.4+ it can be done with Common Table Expressions (CTE). Here is a fast counter with recursive CTE:</p>

<pre>
  def fast_count_comments(story)
    comment_id = story.comments.first&.id
    return 0 unless comment_id

    sql = <<-SQL
      WITH RECURSIVE story_comments AS (
        SELECT id, commentable_id FROM comments WHERE commentable_id = #{comment_id} AND commentable_type = 'Story'
        UNION
        SELECT c.id, c.commentable_id FROM comments AS c
        INNER JOIN story_comments AS sc ON sc.id = c.commentable_id
      ) SELECT count(*) FROM story_comments;
    SQL
    count = Comment.count_by_sql(sql)
    count
  end
</pre >

<p>The expression consists of 5 parts:</p>
<ol>
  <li>Non-recursive subquery:
    <pre>SELECT id, commentable_id FROM comments WHERE commentable_id = #{comment_id} AND commentable_type = 'Story'</pre>
  </li>
  <li>Recursive subquery:
    <pre>SELECT c.id, c.commentable_id FROM comments AS c <br />INNER JOIN story_comments AS sc ON sc.id = c.commentable_id</pre>
  </li>
  <li>Recursively repeat step 2</li>
  <li>Combine results with UNION</li>
  <li>Count results with <pre>SELECT count(*) FROM story_comments</pre></li>
</ol>

<p>Recursive CTE implementaion is much faster. The <code>fast_count_comments</code> method makes 2 sql queries (~20ms with rendering). Here is the output from the rack mini profiler:</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/fast_query.png" alt="fast profiler" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-02-01/fast_query_details.png" alt="fast profiler" />
  </div>
</div>
