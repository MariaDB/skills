# Pagination: Cursor-Based Instead of OFFSET

`OFFSET` is a hidden performance trap. `LIMIT 10 OFFSET 50000` scans and discards 50,000 rows on every page load.

```sql
-- ✗ Slow — scans 50,000 rows to skip them:
SELECT id, title FROM posts ORDER BY id DESC LIMIT 10 OFFSET 50000;

-- ✅ Fast — index seek directly to the cursor position:
-- First page:
SELECT id, title FROM posts ORDER BY id DESC LIMIT 10;

-- Next page (pass last id from previous result as $last_id):
SELECT id, title FROM posts WHERE id < $last_id ORDER BY id DESC LIMIT 10;
```

For filtered queries, include the filter column in the index alongside id:

```sql
-- Query: WHERE category = 'news' ORDER BY id DESC
CREATE INDEX idx_cat_id ON posts (category, id);
-- Cursor query:
SELECT id, title FROM posts WHERE category = 'news' AND id < $last_id ORDER BY id DESC LIMIT 10;
```

To detect whether another page exists, fetch `LIMIT 11` and check if the 11th row appears.

### What LLMs Get Wrong
- `SELECT * FROM table LIMIT 10 OFFSET 50000`: Use cursor-based pagination — `OFFSET` scans all skipped rows.

### Quick Wins
4. Check for `OFFSET` in pagination queries.
