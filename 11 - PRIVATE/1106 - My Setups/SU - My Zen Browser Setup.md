- Set as default browser
- Set default search engine `DuckduckGo`
- Change look and feel to Collapsed Sidebar in Preferences
## Setup `about:config`
- Set `zen.urlbar.replace-newtab` to `false`
- Set `zen.view.show-newtab-button-top` to `false`
- Set `zen.window-sync.enabled` to `false`
## Customize toolbar
- Home between Backward and Foreword buttons
- Open Settings on top left edge
- Downloads at bottom left edge
- Expend/Collapse toolbar at bottom right edge

## 2. Setup Zen Mods
```url
about:preferences#zenMarketplace
```
## 3. Setup sine mods
- Download `sine-linux-x64`
```url
https://github.com/CosmoCreeper/Sine/releases/
```
- Install in (release) profile
```bash
sudo chown -R $USER:$USER /opt/zen-browser-bin/
sudo chmod -R u+w /opt/zen-browser-bin/

chmod +x sine-linux-x64
sudo ./sine-linux-x64

sudo chown -R root:root /opt/zen-browser-bin/


sudo chown -R $USER:$USER /opt/zen-browser-bin/
./sine-linux-x64  # Run WITHOUT sudo
```
## 4. Set extensions
- Dark Reader 
- Zen Internet
- uBlock Origin 
- Bonjourr · Minimalist Startpage
- Proton Pass: Free Password Manager
- AB Download Manager Browser Integration
	- Add extensions
- Privacy Badger 
- TWP - Translate Web Pages 
- Containerise
- Grayscale Browsing
- Ultra Volume Booster
- dogment PDF Reader
- Snap Links
- Buster: Captcha Solver for Humans
- Proton VPN: Fast & Secure
- Obsidian Web Clipper
- Surge

- Cookie-Editor
- Copy Frame or Page URL
- FastForward
- FastStream Video Player
- Find Font
- Font Changer
- Font Changer
- FontFinder
- Grammarly: Al Writing and Grammar Checker App
- Medium Parser
- Mobile View Switcher
- OCR - Image Reader
- P-Stream extension
- Progressive Web Apps for Firefox
- Responsive Viewer
- Search by Image 
- Tranquility Reader 
- Video DownloadHelper 
- Violentmonkey
- VK Next - functions for VK
- Watch2Gether
- Wayback Machine
- Web Archives 
- YouTube NonStop 

## Spaces
- WORK
	- Folders
		- AI
- PLAY
- Z
## Containers 
- Default
- right9code
- right9postbox
- abhi9tiyal
- 
## 5. Bookmark Folders

### Bookmark Toolbar
- 01 - Sailing Seas
	- 00 - Catalogue
	- 



```bash
{
  "about": {
    "browser": "firefox",
    "version": "22.0.0"
  },
  "showall": true,
  "lang": "en",
  "dark": "enable",
  "favicon": "",
  "tabtitle": "",
  "greeting": "",
  "greetingsize": "3",
  "greetingsmode": "auto",
  "pagegap": 0,
  "pagewidth": 2200,
  "time": true,
  "main": true,
  "dateformat": "auto",
  "quicklinks": true,
  "textShadow": 0.2,
  "announcements": "major",
  "review": -1,
  "css": "@import url('https://fonts.googleapis.com/css2?family=Comfortaa:wght@300&display=swap');\n\nbody, h1, h2, h3, h4, h5, h6, p, span, div {\n    font-family: 'Comfortaa', sans-serif !important;\n    font-weight: 300 !important;\n    letter-spacing: 0.015em;\n    font-smooth: always;\n    -webkit-font-smoothing: antialiased;\n    -moz-osx-font-smoothing: grayscale;\n    transition: color 0.3s ease, text-shadow 0.3s ease;\n}\n\n/* Light mode */\n@media (prefers-color-scheme: light) {\n    body, h1, h2, h3, h4, h5, h6, p, span, div {\n        color: #222222;\n        text-shadow: 0 0 1px rgba(0, 0, 0, 0.15);\n    }\n}\n\n/* Dark mode */\n@media (prefers-color-scheme: dark) {\n    body, h1, h2, h3, h4, h5, h6, p, span, div {\n        color: #e0e0e0;\n        text-shadow: 0 0 1px rgba(255, 255, 255, 0.2);\n    }\n}\n\nh1 {\n    font-weight: 400 !important;\n    letter-spacing: 0.025em;\n}\n\np {\n    font-weight: 300 !important;\n    line-height: 1.6;\n    letter-spacing: 0.015em;\n}\n#background {\n    background-color: transparent !important;\n}\nbackground {background-image: none !important; background-color: transparent !important;}\n   .tabbing {background-color: transparent !important;} \n    body {background-color: transparent !important;}",
  "hide": {
    "settingsicon": true,
    "greetings": true,
    "weatherdesc": true,
    "weathericon": false
  },
  "linkstyle": "inline",
  "linktitles": false,
  "linkbackgrounds": false,
  "linknewtab": false,
  "linksrow": "8",
  "linkiconradius": 1.1,
  "linkgroups": {
    "on": true,
    "selected": "default",
    "groups": [
      "default"
    ],
    "pinned": [],
    "synced": []
  },
  "backgrounds": {
    "type": "files",
    "fadein": 600,
    "blur": 36,
    "bright": 0.2,
    "frequency": "pause",
    "color": "#185A63",
    "urls": "",
    "images": "unsplash-images-search",
    "videos": "bonjourr-videos-daylight",
    "queries": {
      "unsplash-images-search": "gruvbox"
    },
    "texture": {
      "type": "none"
    },
    "pausedImage": {
      "color": "#0c2659",
      "urls": {
        "full": "https://images.unsplash.com/photo-1727091462554-a36e6136b6ea?ixid=M3w4ODczOHwwfDF8YWxsfHx8fHx8fHx8MTc1OTU5NjI4N3w&ixlib=rb-4.1.0&auto=format&fit=crop&crop=entropy&h=1080&w=1920&q=80",
        "medium": "https://images.unsplash.com/photo-1727091462554-a36e6136b6ea?ixid=M3w4ODczOHwwfDF8YWxsfHx8fHx8fHx8MTc1OTU5NjI4N3w&ixlib=rb-4.1.0&auto=format&fit=crop&crop=entropy&h=540&w=960&q=50",
        "small": "https://images.unsplash.com/photo-1727091462554-a36e6136b6ea?ixid=M3w4ODczOHwwfDF8YWxsfHx8fHx8fHx8MTc1OTU5NjI4N3w&ixlib=rb-4.1.0&auto=format&fit=crop&crop=entropy&h=108&w=192&q=60"
      },
      "format": "image",
      "page": "https://unsplash.com/photos/the-moon-in-a-clear-blue-sky-with-no-clouds-74YqCNdFYUU",
      "download": "https://unsplash.com/photos/74YqCNdFYUU/download?ixid=M3w4ODczOHwwfDF8YWxsfHx8fHx8fHx8MTc1OTU5NjI4N3w",
      "username": "rd421",
      "name": "R.D. Smith",
      "exif": {
        "name": "SONY, ILCE-7M4",
        "make": "SONY",
        "model": "ILCE-7M4",
        "exposure_time": "1/100",
        "aperture": "6.3",
        "focal_length": "600.0",
        "iso": 80
      }
    }
  },
  "clock": {
    "size": 1.375,
    "ampm": false,
    "analog": true,
    "seconds": true,
    "ampmlabel": false,
    "worldclocks": false,
    "timezone": "auto"
  },
  "worldclocks": [],
  "analogstyle": {
    "face": "braun",
    "hands": "braun",
    "shape": "round",
    "border": "#ffff",
    "background": "#fff3"
  },
  "weather": {
    "city": "dehradun",
    "unit": "metric",
    "provider": "",
    "moreinfo": "none",
    "forecast": "auto",
    "temperature": "actual",
    "geolocation": "approximate"
  },
  "greetingscustom": {
    "morning": "",
    "afternoon": "",
    "evening": "",
    "night": ""
  },
  "notes": {
    "on": false,
    "width": 40,
    "opacity": 0.1,
    "align": "left",
    "text": "## Edit this note!\n\n[ ] With markdown titles, lists, and checkboxes\n\n[ ] Learn more on https://bonjourr.fr/docs/overview"
  },
  "searchbar": {
    "on": true,
    "opacity": 0.1,
    "newtab": false,
    "suggestions": true,
    "engine": "google",
    "request": "",
    "placeholder": ""
  },
  "quotes": {
    "on": false,
    "type": "classic",
    "frequency": "day",
    "author": false,
    "last": 1769453825571
  },
  "pomodoro": {
    "on": false,
    "mode": "pomodoro",
    "pause": 0,
    "end": 0,
    "timeFor": {
      "pomodoro": 1500,
      "break": 300,
      "longbreak": 1200
    },
    "focus": false,
    "sound": true,
    "history": []
  },
  "font": {
    "size": "14",
    "family": "Fragment Mono",
    "system": true,
    "weightlist": [],
    "weight": "400"
  },
  "supporters": {
    "enabled": false,
    "closed": true,
    "month": 3
  },
  "move": {
    "selection": "single",
    "layouts": {
      "single": {
        "items": {
          "time": {
            "text": "",
            "box": "end center"
          },
          "main": {
            "text": "",
            "box": "baseline end"
          },
          "quicklinks": {
            "text": "",
            "box": "end center"
          },
          "searchbar": {
            "text": "",
            "box": "baseline center"
          }
        },
        "grid": [
          [
            "main"
          ],
          [
            "time"
          ],
          [
            "searchbar"
          ],
          [
            "quicklinks"
          ]
        ]
      },
      "triple": {
        "items": {
          "time": {
            "text": "",
            "box": "end center"
          }
        },
        "grid": [
          [
            "time",
            ".",
            "."
          ],
          [
            "main",
            ".",
            "."
          ],
          [
            "quicklinks",
            ".",
            "."
          ]
        ]
      }
    }
  },
  "linkseojheg": {
    "_id": "linkseojheg",
    "parent": "default",
    "order": 9,
    "title": "aistudio",
    "url": "https://aistudio.google.com/prompts/new_chat",
    "icon": {
      "type": "url",
      "value": "blob:https://lobehub.com/9218a7b1-ff55-4ad1-875a-b8016cceec46"
    }
  },
  "linksocqlgb": {
    "_id": "linksocqlgb",
    "parent": "default",
    "order": 8,
    "title": "Koofr",
    "url": "https://app.koofr.net/app/"
  },
  "linksarcajh": {
    "_id": "linksarcajh",
    "parent": "default",
    "order": 13,
    "title": "YT",
    "url": "https://www.youtube.com/"
  },
  "linksabkdra": {
    "_id": "linksabkdra",
    "parent": "default",
    "order": 11,
    "title": "deepseek",
    "url": "https://chat.deepseek.com/"
  },
  "linksherfnp": {
    "_id": "linksherfnp",
    "parent": "default",
    "order": 6,
    "title": "perplexity",
    "url": "https://www.perplexity.ai/",
    "icon": {
      "type": "auto",
      "value": "https://services.bonjourr.fr/favicon/blob/https://www.perplexity.ai/?r=1743346730113"
    }
  },
  "linksherpji": {
    "_id": "linksherpji",
    "parent": "default",
    "order": 5,
    "title": "gemini",
    "url": "https://gemini.google.com",
    "icon": {
      "type": "auto",
      "value": "https://services.bonjourr.fr/favicon/blob/https://gemini.google.com?r=1747774926802"
    }
  },
  "linksfapijp": {
    "_id": "linksfapijp",
    "parent": "default",
    "order": 2,
    "title": "docs",
    "url": "https://docs.google.com/document/u/0/"
  },
  "linkskncaoe": {
    "_id": "linkskncaoe",
    "parent": "default",
    "order": 4,
    "title": "drive",
    "url": "https://drive.google.com"
  },
  "linksnpeigc": {
    "_id": "linksnpeigc",
    "parent": "default",
    "order": 12,
    "title": "DAB Music",
    "url": "https://dabplayer.vercel.app/"
  },
  "linkslccmdh": {
    "_id": "linkslccmdh",
    "parent": "default",
    "order": 1,
    "title": "sheets",
    "url": "https://docs.google.com/spreadsheets/u/0/",
    "icon": {
      "type": "url",
      "value": "https://www.gstatic.com/images/branding/product/2x/sheets_2020q4_48dp.png"
    }
  },
  "linkspnajdg": {
    "_id": "linkspnajdg",
    "parent": "default",
    "order": 3,
    "title": "mail",
    "url": "https://mail.google.com"
  },
  "linksaapjnj": {
    "_id": "linksaapjnj",
    "parent": "default",
    "order": 10,
    "title": "miramax",
    "url": "https://agent.minimax.io/"
  },
  "linksajodme": {
    "_id": "linksajodme",
    "parent": "default",
    "order": 7,
    "title": "qwen",
    "url": "https://chat.qwen.ai/"
  },
  "linksrrkgpf": {
    "_id": "linksrrkgpf",
    "parent": "default",
    "order": 0,
    "title": "keep",
    "url": "https://keep.google.com/"
  },
  "background_solid": "#222"
}
```


