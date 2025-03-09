# Travel Social Network - System Design

Example of Social Network System Requirements

## Functional Requirements
- Users of our system are travellers
- `User` could publish the posts from travel
    - `Post`:
        - message
        - photos
        - geolocation
- `User` could comment & rate of posts and of other users
    - `Comment`:
        - message
    - `Rating`
        - stars
- `User` could follow other users and vise versa
    - User has `Profile` feed with list of own posts
    - User has followers `Feed` with posts of followers
- `User` could see popular `Spot` list and posts from those spots
    - `Spot` list aggregated by near geolocations (500m around)
- `User` could name `Spot` or choose `Spot` name of other `User`

## Non-Functional Requirements
- Audience
    - 10 000 000 DAU, linear grow in 1 year
    - Region: CIS only
- `User` activity
    - `Post`
        - creates 1 `Post` per week, 3 photo per post
        - views `Feed` 2 times per day, makes 10 requests per time, load 20 `Posts` per request
    - `Comment`
        - writes 3 `Comment` per day
        - read 5 `Post` comments per day, makes 5 requests per `Post`, load 15 `Comment` per request
    - `Rating`
        - read `Rating`, 1-1 with `Post` so same rate
        - rates `Post` 5 times per day
    - `Spot`
        - read `Spot` list 1 time per week
- Data storage
    - data should be saved forever
- Limits
    - `Post`
        - avg. 3 img per post, max 10 img per post
    - `Comment`
        - 1k comments per `Post`
    - Followers
        - avg `User` could have up to 100k followers
        - some popular users could have 1kk followers

    - Availability 99.95%
    - Performance
        - load a `Feed` <1s
        - create a `Post` <1s
        - create `Comment` <1s
        - rate the `Post` <500ms
    - Seasonality
        - 2 times per year before winter holidays/summer we could expect grow up to 50% DAU
        - 2 times per year spring/autumn we could expect grow 30% DAU

## Calculations

### RPS
- `Post`
    - read:  10kk DAU * 2 per day * 10 req per time /86400 = 2k RPS
    - write: 10kk DAU * 1 post per week/ 7 (days) / 86400 secs = 14 RPS
- `Comment`
    - read:  10kk DAU * 5 per day * 5 req per post /86400 = 2.5k RPS
    - write: 10kk DAU * 3 per day / 86400 secs = 300 RPS
- `Rating`
    - read:  10kk DAU * 2 per day * 10 req per time /86400 = 2k RPS
    - write: 10kk DAU * 5 per day / 86400 secs = 500 RPS
- `Spot`
    - read:  10kk DAU / 7 days / 86400 = 14 RPS

### Data size
- `Post`
    - id - 8 bytes
    - message - 1024 bytes
    - geolocation - 2 (lon, lat) * 8 bytes = 16 bytes
    - img id - 255 bytes
    - user_id - 8 bytes
    - created_at - 8 bytes
    - updated_at - 8 bytes
  ```
  Total per `Post` (data): 8 + 1024 + 16 + 255 + 8 + 8 + 8 = 1400 bytes
  Total per `Post` (media): 1MB
  ```
- `Comment`
    - id - 8 bytes
    - message - 255 bytes
    - post_id - 8 bytes
    - user_id - 8 bytes
    - created_at - 8 bytes
    - updated_at - 8 bytes
  ```
  Total per `Comment`: 8 * 5 + 255 = 300 bytes 
  ```
- `Rating`
    - id - 8 bytes
    - post_id - 8 bytes
    - user_id - 8 bytes
    - starts - 1 byte
    - created_at - 8 bytes
  ```
  Total per `Rating`: 8 * 4 = 33 bytes
  ```
- `Spot`
    - id - 8 bytes
    - name - 50 bytes
    - created_at - 8 bytes
  ```
  Total per `Spot`: 8 * 2 + 50 = 66 bytes
  ```
### Traffic

`Post`

    read (data):   2k RPS * 10 post per request * 1400 bytes = 28 MB/s 
    write (data):  14 RPS * 1400 bytes = 19KB/s

    read (media):  2k RPS * 10 post per request * 1 MB * 3 img per post = 58GB/s
    write (media): 14 RPS * 1 MB * 3 per post = 42MB/s

`Comment`

    read:  2.5k RPS * 15 per req * 300 bytes = 10MB/s
    write: 300 RPS * 5 per comment = 1KB/s

`Rating`

    read:  2k RPS  * 33 bytes = 64KB/s
    write: 500 RPS * 33 bytes = 16KB/s

`Spot`

    read:  14 RPS  * 66 bytes * 10 sports per requests = 10KB/s

Total (data)

    read   28 MB/s + 10MB/s + 64KB/s + 10KB/s = 40 MB/s
    write: 19KB/s +  1KB/s + 16KB/s = 36KB/s

Total (media)

    read:  58GB/s
    write: 42MB/s


### Connections
    10kk DAU * 0.1 (~10% of DAU) = 1 000 000 connections