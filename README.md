# whatlastgenre

Improves genre metadata of audio files based on tags from various music sites.

* Supported audio files: flac, ogg, mp3, m4a
* Supported music sites: What.CD, Last.FM, MusicBrainz, Discogs, EchoNest, Idiomag
* Feature Overview
  * Gets genre tags for artists and albums from music sites and finds the most
  eligible ones by merging, splitting, filtering and scoring them.
    * Merges similar tags in different writings to ensure consistent names, eg.
    DnB, D&B, Drum and Bass -> Drum & Bass;
    Alt., Altern, Alterneitif -> Alternative
    * Splits tags in various applicable ways, eg.
    Jazz/Funk&Rock -> Jazz, Funk, Rock; Alternative Rock -> Alternative, Rock
    * Filters crappy tags without using a whitelist
    * Scores tags while taking personal preferences into account
  * Caches all data received from music sites to make reruns super fast
  * Makes use of MusicBrainz IDs when possible
  * Optional: gets release type (Album, EP, Anthology, ...) from What.CD
  * Optional: gets MusicBrainz IDs
  * Interactive mode to select right MBID and releasetype for ambiguous results
  * Dry-mode for safe testing


## How it works
It scans through folders for albums and receives genre tags for them and their
artists from different music sites. Equal tags in different writings will be
split and/or merged to ensure consistent names and proper scoring. The tags
get filtered and scored. Artist and album tags get handled separately and then
merged using configurable score modifiers. The best scored tags will be saved
as genre metadata in the corresponding album tracks. All data received from
music sites gets cached so that rerunning the script will be super fast. There
are several score multipliers to adjust the scoring to your needs and take your
personal preferences into account. Please take a look at "Configuration options
explained" below for more details.

##### Tag scoring with count (What.CD, Last.FM, MusicBrainz, Idiomag)
If counts are supplied for the tags they will get scored by `count/topcount`,
where `topcount` is the highest count of all tags from a source. So the top
tag gets a score of 1.0, a tag having only half of the top tag's count gets
a score of 0.5 and so on.

##### Tag scoring without count (What.CD, Discogs, EchoNest, Idiomag)
Tags supplied without a count will be scored `0.85^(n-1)`, where `n` is the
total number of tags supplied by this source. The more tags the lower the score
for each tag will be. So if only one tag is supplied, it will get a score of
1.0, two tags will get a score of 0.85 each, three get 0.72 each and so on...

##### Tag merging of artist and album tags
After all tags have been gathered the scores of album and artist tags will be
scaled and then merged together taking the artist score modifier option into
account. This enables that multiple albums from one artist get more equal tags
by giving tags found by artist searches advantage over tags found by album
searches.

See `artist` score option below.

##### Tag scoring for various artist albums without any specific album artist
If there is no specific album artist for a various artist album every track
artist gets used for searching. Tags for artists that appear multiple times in
that album get counted multiple times.

See `various` score option below.


## Installation
You'll need Python 2.7.

Install dependencies with your package manager, on Debian based distros run
this as root:

    apt-get install python-mutagen python-requests

* Alternatively, install dependencies by using python-pip:
`pip install mutagen requests`
* Clone the git or download and unzip the [source package]
(http://github.com/YetAnotherNerd/whatlastgenre/archive/master.zip)
* Run it without install by using `./whatlastgenre` from the directory you
cloned/extracted to
* Install it by running `python setup.py install` as root in that directory

##### Optional Dependencies
* `oauth2` is required for Discogs (disabled by default). If you want to use
Discogs, use `apt-get install python-oauth2` or `pip install oauth2` to
install it (like above)


## Configuration
A configuration file with default values will be created on first run.

The default location for the configuration file is `~/.whatlastgenre/config`.

### Example configuration file
```
[wlg]
sources = whatcd, discogs, mbrainz, lastfm
cache_timeout = 60
whatcduser = whatusername
whatcdpass = whatpassword
[genres]
love = soundtrack
hate = alternative, electronic, indie, pop, rock
blacklist = charts, male vocalist, other
filters = instrument, label, location, year
[scores]
artist = 1.33
various = 0.66
splitup = 0.33
src_whatcd = 1.66
src_lastfm = 0.66
src_mbrainz = 1.00
src_discogs = 1.00
src_idiomag = 1.00
src_echonest = 1.00
```

### Configuration options explained

#### whatlastgenre (wlg) section

##### sources option
The music sites where to get the genre tags from.
* `whatcd` [[URL](https://what.cd)]
well-kept tags from community
* `lastfm` [[URL](http://last.fm)]
mbid search possible, many personal tags from users
* `mbrainz` [[URL](http://musicbrainz.org)]
home of mbids
* `discogs` [[URL](http://discogs.com)]
album only, fixed list of [genres]
(http://discogs.com/help/submission-guidelines-release-genres-styles.html) and
[styles](http://wiki.discogs.com/index.php/Style_Guide),
now requires authentication (own account needed)
* `idiomag` [[URL](http://idiomag.com)]
artist only
* `echonest` [[URL](http://echonest.com)]
artist only, fixed list of
[genres](http://developer.echonest.com/docs/v4/artist.html#list-genres)

##### cache_timout option
Time in days after which cache hits get invalid.

Default `30`, Range `14 - 180`

#### genres section

##### love and hate options
List of tags that get a multiplier bonus of `2.0` and `0.5` respectively.
Should be considered as "soft white-/blacklist" where you can in-/decrease the
occurrence of specific tags that you don't like or that are too inaccurate for
you without fully banning them like with the blacklist option.

##### filters option
Use this to activate genre tag filters for specific groups of tags. The filter
names relate to the filter sections in the tags.txt file.
* `instrument` instrument related names, like piano or guitarist
* `label` label names
* `location` country, city and nationality names
* `year` year tags, like 2010, 80s

#### scores section

##### artist option
Score multiplier to give tags found by albumartist searches advantage over tags
from album searches. The tags get stored separately at first but then put
together while taking this multiplier into account. This enables that multiple
albums from one artist get more equal tags.

`<artist tags> * <artist score> + <album tags>`

Default `1.33`, Range `0.5 - 2.0`
* `< 1.0` prefer album tags
* `= 1.0` handle them equally
* `> 1.0` prefer artist tags

##### various option
Score multiplier similar to artist option, but this one applies to various
artists releases if there is no albumartist and all the track artists get used
for searching. For example: a 5 track va-album with 3 tracks from artist A and
2 tracks from artist B will get tags like this:

`(3 * <artist A tags> + 2 * <artist B tags>) * <various score> + <album tags>`

Default `0.66`, Range `0.1 - 1.0`
* `< 1.0` prefer album tags
* `= 1.0` handle them equally

##### splitup option
Score multiplier for modifying the score of the base tag from a tag that got
split up by space, this enables you to decide whether to keep, prefer or ban
the base tags. For example, lets say we have 'Alternative Rock' with a score
of 1: It will end up as Alternative with score 1, Rock with score 1 and
Alternative Rock with score `1 * <splitup-score>`. So if you don't want to keep
Alternative Rock, just set it to 0, but consider using a very small number
instead to avoid banning them totally.

Default `0.33`, Range `0.0 - 1.0`
* `= 0.0` forget about the "base" tags
* `< 1.0` prefer split parts
* `= 1.0` handle them equally

##### src_* options
Every source has its own score multiplier, so music sites that generally
provide higher quality tags can be given advantage over sources that often
provide bad, inaccurate or personal tags. Increase if you trust the tags from
a source, lower if the source provides many inaccurate or personal tags. If you
don't want tags from a specific source remove it from the sources list option.

Default `1.0`, Range `0.5 - 2.0`. See sources option above.


## Usage
```
usage: whatlastgenre [-h] [-v] [-n] [-i] [-c] [-r] [-m] [-l N] path [path ...]

positional arguments:
  path                 folder(s) to scan for albums

optional arguments:
  -h, --help           show this help message and exit
  -v, --verbose        verbose output (-vv for debug) (default: 0)
  -n, --dry            don't save metadata (default: False)
  -i, --interactive    interactive mode (default: False)
  -c, --no-cache       bypass cache hits (default: False)
  -r, --tag-release    tag release type (from What.CD) (default: False)
  -m, --tag-mbids      tag musicbrainz ids (default: False)
  -l N, --tag-limit N  max. number of genre tags (default: 4)
```

If you seriously want to tag release-types `-r` or musicbrainz-ids `-m` you
should also enable interactive mode `-i`. Consider to save the MBIDs `-m` when
using mbrainz, you searched for them, why not save them? ;)

I recommend first doing a dry-run to fill the cache and then doing a normal
run with interactivity enabled. This way you can answer all interactivity
questions without much waiting time in between.

Remove the cache file to reset the cache. If you only want to reset the cache
for specific albums rerun with `-c` on this albums to ignore cache hits.

### Examples

Do a verbose dry-run on your albums in /home/user/music changing nothing:

	$ whatlastgenre -vn /home/user/music

Tag max. 3 genre tags for all albums in /home/user/music:

	$ whatlastgenre -l 3 /home/user/music

To get the most of it for all albums in /home/user/music and /media/music:

	$ whatlastgenre -irml 5 /home/user/music /media/music



whatlastgenre doesn't correct any other tags. If your music files are badly or
not tagged it won't work well at all.

Please report any bugs and errors you encounter, i would like to fix them :)

