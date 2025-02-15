### Files:

#### index.js
```js
const fs = require("fs");
const readline = require("readline");
const { google } = require("googleapis");

async function run() {
    const oAuth2Client= new google.auth.OAuth2({
        ClientId:"",
        ClientSecret:"",
        RedirectUri: "urn:ietf:wg:oauth:2.0:oob", 
    });

    const authUrl= oAuth2Client.generateAuthUrl({
        scope: "https://www.googleapis.com/auth/youtube",
        prompt: "consent",
        access_type: "offline",
    });
    const rl= readLine.createInterface({
        input: process.stidin,
        output: process.stdout,
    });
    console.log("Go to the following link in your browser then type the " +
    "authorization code:");

    console.log(authUrl);
    const code= await new Promise((resolve) => {
        rl.question("Enter the authorization code: ", (code) => {
            resolve(code);
            rl.close();
        });
    });
    const token= await oAuth2Client getToken(code);
    fs.writeFile(tokenPath, JSON.stringify(token), (err) => {
        if(err) return console.error(err);
        console.log("Token stored to " + tokenPath);
    })
}
run();
```

#### verify.js
```js
const {google} = require("googleapis");
const fs = require("fs");

async function run() {
    const content = fs.readFileSync('credentials.json');
    const credentials = JSON.parse(content);

    const {client_secret, client_id, redirect_uris} = credentials.installed;
    const oAuth2Client = new google.auth.OAuth2(client_id, client_secret,redirect_uris[0]);
    const token = fs.readFileSync(tokenPath);
    oAuth2Client.setCredentials(JSON.parse(token));
    const service = google.youtube({
        version: "v3",
        auth: oAuth2Client,
    });
    const res = await service.channels.list({
        mine: true,
        part: "snippet, id",
    });
    const channelId= res.data.items[0].id;
    const response = await service.channels.list({
        id: channelId,
        part: "contentDetails",
    }); 
    const uploadsId= response.data.items[0].contentDetails.relatedPlaylists
    .uploads;
    const videos = await service.playlistItems.list({
        playlistId: uploadsId,
        part: "snippet",
        maxResults: 50,
    });
    videos.data.items.forEach(item => {
        console.log(`${item.snippet.title} (${item.snippet.resourceId.videoId})`)
    })
}
run();
```