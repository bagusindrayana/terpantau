# Web Untuk Stream live CCTV

## Rest API
- Data CCTV : https://github.com/bagusindrayana/scrape-cctv-gov-id
- Thumbnail CCTV : https://github.com/bagusindrayana/get-thumbnail-api
- Proxy : https://github.com/bagusindrayana/m3u8_proxy-cors
- Live Chat (Pusher) (Optional) : https://github.com/bagusindrayana/livechat-pusher

### Set Rest API
- set all rest api url in `env.example` and rename to `.env`
```env
PUBLIC_API_URL=http://localhost:8000
PUBLIC_API_CORS=http://localhost:5010/cors?url=
PUBLIC_API_LIVECHAT=http://localhost:5197
PUBLIC_API_THUMBNAIL="http://localhost:8111/get?video_url="
```