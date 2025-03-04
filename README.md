# 📌 aiopinboard: A Python 3 Library for Pinboard

[![CI][ci-badge]][ci]
[![PyPI][pypi-badge]][pypi]
[![Version][version-badge]][version]
[![License][license-badge]][license]
[![Code Coverage][codecov-badge]][codecov]
[![Maintainability][maintainability-badge]][maintainability]

<a href="https://www.buymeacoffee.com/bachya1208P" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

`aiopinboard` is a Python3, `asyncio`-focused library for interacting with the
[Pinboard][pinboard] API.

- [Installation](#installation)
- [Python Versions](#python-versions)
- [API Token](#api-token)
- [Usage](#usage)
  - [Bookmarks](#bookmarks)
    - [The `Bookmark` Object](#the--bookmark--object)
    - [Getting the Last Change Datetime](#getting-the-last-change-datetime)
    - [Getting Bookmarks](#getting-bookmarks)
    - [Adding a Bookmark](#adding-a-bookmark)
    - [Deleting a Bookmark](#deleting-a-bookmark)
  - [Tags](#tags)
    - [Getting Tags](#getting-tags)
    - [Getting Suggested Tags](#getting-suggested-tags)
    - [Deleting a Tag](#deleting-a-tag)
    - [Renaming a Tag](#renaming-a-tag)
  - [Notes](#notes)
    - [The `Note` Object](#the--note--object)
    - [Getting Notes](#getting-notes)
- [Contributing](#contributing)

# Installation

```bash
pip install aiopinboard
```

# Python Versions

`aiopinboard` is currently supported on:

- Python 3.10
- Python 3.11
- Python 3.12

# API Token

You can retrieve your Pinboard API token via
[your account's settings page][pinboard-settings].

# Usage

`aiopinboard` endeavors to replicate all of the endpoints in
[the Pinboard API documentation][pinboard-api] with sane, usable responses.

All API usage starts with creating an `API` object that contains your Pinboard API token:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    # do things!


asyncio.run(main())
```

## Bookmarks

### The `Bookmark` Object

API endpoints that retrieve one or more bookmarks will return `Bookmark` objects, which
carry all of the expected properties of a bookmark:

- `hash`: the unique identifier of the bookmark
- `href`: the bookmark's URL
- `title`: the bookmark's title
- `description`: the bookmark's description
- `last_modified`: the UTC date the bookmark was last modified
- `tags`: a list of tags applied to the bookmark
- `unread`: whether the bookmark is unread
- `shared`: whether the bookmark is shared

### Getting the Last Change Datetime

To get the UTC datetime of the last "change" (bookmark added, updated, or deleted):

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    """Run!"""
    api = API("<PINBOARD_API_TOKEN>")
    last_change_dt = await api.bookmark.async_get_last_change_datetime()
    # >>> datetime.datetime(2020, 9, 3, 13, 7, 19, tzinfo=<UTC>)


asyncio.run(main())
```

This method should be used to determine whether additional API calls should be made –
for example, if nothing has changed since the last time a request was made, the
implementing library can halt.

### Getting Bookmarks

To get a bookmark by its URL:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_get_bookmark_by_url("https://my.com/bookmark")
    # >>> <Bookmark href="https://my.com/bookmark">


asyncio.run(main())
```

To get all bookmarks

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_get_all_bookmarks()
    # >>> [<Bookmark ...>, <Bookmark ...>]


asyncio.run(main())
```

You can specify several optional parameters while getting all bookmarks:

- `tags`: an optional list of tags to filter results by
- `start`: the optional starting index to return (defaults to the start)
- `results`: the optional number of results (defaults to all)
- `from_dt`: the optional datetime to start from
- `to_dt`: the optional datetime to end at

To get all bookmarks created on a certain date:

```python
import asyncio
from datetime import date

from aiopinboard import API


async def main() -> None:
    """Run!"""
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_get_bookmarks_by_date(date.today())
    # >>> [<Bookmark ...>, <Bookmark ...>]

    # Optionally filter the results with a list of tags – note that only bookmarks that
    # have all tags will be returned:
    await api.bookmark.async_get_bookmarks_by_date(date.today(), tags=["tag1", "tag2"])
    # >>> [<Bookmark ...>, <Bookmark ...>]


asyncio.run(main())
```

To get recent bookmarks:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_get_recent_bookmarks(count=10)
    # >>> [<Bookmark ...>, <Bookmark ...>]

    # Optionally filter the results with a list of tags – note that only bookmarks that
    # have all tags will be returned:
    await api.bookmark.async_get_recent_bookmarks(count=20, tags=["tag1", "tag2"])
    # >>> [<Bookmark ...>, <Bookmark ...>]


asyncio.run(main())
```

To get a summary of dates and how many bookmarks were created on those dates:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    dates = await api.bookmark.async_get_dates()
    # >>> {datetime.date(2020, 09, 05): 4, ...}


asyncio.run(main())
```

### Adding a Bookmark

To add a bookmark:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_add_bookmark("https://my.com/bookmark", "My New Bookmark")


asyncio.run(main())
```

You can specify several optional parameters while adding a bookmark:

- `description`: the optional description of the bookmark
- `tags`: an optional list of tags to assign to the bookmark
- `created_datetime`: the optional creation datetime to use (defaults to now)
- `replace`: whether this should replace a bookmark with the same URL
- `shared`: whether this bookmark should be shared
- `toread`: whether this bookmark should be unread

### Deleting a Bookmark

To delete a bookmark by its URL:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_delete_bookmark("https://my.com/bookmark")


asyncio.run(main())
```

## Tags

### Getting Tags

To get all tags for an account (and a count of how often each tag is used):

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.tag.async_get_tags()
    # >>> {"tag1": 3, "tag2": 8}


asyncio.run(main())
```

### Getting Suggested Tags

To get lists of popular (used by the community) and recommended (used by you) tags for a
particular URL:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.bookmark.async_get_suggested_tags("https://my.com/bookmark")
    # >>> {"popular": ["tag1", "tag2"], "recommended": ["tag3"]}


asyncio.run(main())
```

### Deleting a Tag

To delete a tag:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.tag.async_delete_tag("tag1")


asyncio.run(main())
```

### Renaming a Tag

To rename a tag:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.tag.async_rename_tag("old-tag", "new-tag")


asyncio.run(main())
```

## Notes

### The `Note` Object

API endpoints that retrieve one or more notes will return `Note` objects, which
carry all of the expected properties of a note:

- `note_id`: the unique ID
- `title`: the title
- `hash`: the computed hash
- `created_at`: the UTC datetime the note was created
- `updated_at`: the UTC datetime the note was updated
- `length`: the length

### Getting Notes

To get all notes for an account:

```python
import asyncio

from aiopinboard import API


async def main() -> None:
    api = API("<PINBOARD_API_TOKEN>")
    await api.note.async_get_notes()
    # >>> [<Note ...>, <Note ...>]


asyncio.run(main())
```

# Contributing

Thanks to all of [our contributors][contributors] so far!

1. [Check for open features/bugs][issues] or [initiate a discussion on one][new-issue].
2. [Fork the repository][fork].
3. (_optional, but highly recommended_) Create a virtual environment: `python3 -m venv .venv`
4. (_optional, but highly recommended_) Enter the virtual environment: `source ./.venv/bin/activate`
5. Install the dev environment: `script/setup`
6. Code your new feature or bug fix on a new branch.
7. Write tests that cover your new functionality.
8. Run tests and ensure 100% code coverage: `poetry run pytest --cov aiopinboard tests`
9. Update `README.md` with any new documentation.
10. Submit a pull request!

[aiohttp]: https://github.com/aio-libs/aiohttp
[ambient-weather-dashboard]: https://dashboard.ambientweather.net
[ambient-weather-rate-limiting]: https://ambientweather.docs.apiary.io/#introduction/rate-limiting
[ambient-weather]: https://ambientweather.net
[ci-badge]: https://github.com/bachya/aiopinboard/workflows/CI/badge.svg
[ci]: https://github.com/bachya/aiopinboard/actions
[codecov-badge]: https://codecov.io/gh/bachya/aiopinboard/branch/dev/graph/badge.svg
[codecov]: https://codecov.io/gh/bachya/aiopinboard
[contributors]: https://github.com/bachya/aiopinboard/graphs/contributors
[fork]: https://github.com/bachya/aiopinboard/fork
[issues]: https://github.com/bachya/aiopinboard/issues
[license-badge]: https://img.shields.io/pypi/l/aiopinboard.svg
[license]: https://github.com/bachya/aiopinboard/blob/main/LICENSE
[maintainability-badge]: https://api.codeclimate.com/v1/badges/4c0360a07493d3c1fd03/maintainability
[maintainability]: https://codeclimate.com/github/bachya/aiopinboard/maintainability
[new-issue]: https://github.com/bachya/aiopinboard/issues/new
[new-issue]: https://github.com/bachya/aiopinboard/issues/new
[pinboard-api]: https://pinboard.in/api
[pinboard-settings]: https://pinboard.in/settings/password
[pinboard]: https://pinboard.in
[pypi-badge]: https://img.shields.io/pypi/v/aiopinboard.svg
[pypi]: https://pypi.python.org/pypi/aiopinboard
[version-badge]: https://img.shields.io/pypi/pyversions/aiopinboard.svg
[version]: https://pypi.python.org/pypi/aiopinboard
