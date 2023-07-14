# Chrome URL autocomplete rank adjustments

Say, hypothetically, that there are two urls you often type into the url bar that start with the same letter, like news.ycombinator.com and nytimes.com. You visit both of them at roughly the same frequency so chrome goes back and forth between which one it suggests after typing the first 'n'.

You might like chrome to behave consistently. So here's how to adjust the scoring.

In your chrome profile directory (`~/.config/chromium/Default`) there's a sqlite db called History. In it there's a table called `urls`:
```
CREATE TABLE urls(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  url LONGVARCHAR,
  title LONGVARCHAR,
  visit_count INTEGER DEFAULT 0 NOT NULL,
  typed_count INTEGER DEFAULT 0 NOT NULL,
  last_visit_time INTEGER NOT NULL,
  hidden INTEGER DEFAULT 0 NOT NULL
);
```

Adjust the `typed_count` on the urls that are conflicting to further distance them:

```
sqlite> SELECT * from urls where url = 'https://news.ycombinator.com/' or url = 'https://www.nytimes.com/';
             id = 59
            url = https://news.ycombinator.com/
          title = Hacker News
    visit_count = 954
    typed_count = 857
last_visit_time = 13333818754933943
         hidden = 0

             id = 69
            url = https://www.nytimes.com/
          title = The New York Times - Breaking News, US News, World News and Videos
    visit_count = 1066
    typed_count = 850
last_visit_time = 13333819319008480
         hidden = 0

sqlite> UPDATE urls set typed_count = 600 where id = 69;
```
