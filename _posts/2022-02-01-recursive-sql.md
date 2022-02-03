sql = <<-SQL
WITH RECURSIVE story_comments AS (
SELECT id, commentable_id FROM comments WHERE commentable_id = #{comment_id} AND commentable_type = 'Story'
UNION
SELECT c.id, c.commentable_id FROM comments AS c
INNER JOIN story_comments AS sc ON sc.id = c.commentable_id
) SELECT count(\*) FROM story_comments;
SQL
