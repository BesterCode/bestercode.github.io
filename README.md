## What is it?
This is a Vue 3 + Vite project with a configured Action to automatically build and deploy to Github Pages.\
Live version of the website: [NES](https://bestercode.github.io/#/nes) & [Sega](https://bestercode.github.io/#/sega)

## Running locally
```console
git pull
nvm use v18.3.0 # in case you're on something older, like v14.5.0
npm install
npm run dev
```

## Credits
The first half of the Sega titles is an arcade connoisseur list composed by [Nutmeg](https://rpgcodex.net/forums/members/nutmeg.16903/)\
The second half consists of mass audience oriented games by the author of this repo.\
The same goes for the NES titles.\
Thanks to Igor for the Famicom cartridge design.\
To contribute, simply add something to [sega.yaml](https://github.com/BesterCode/bestercode.github.io/blob/master/src/assets/sega.yaml) or [nes.yaml](https://github.com/BesterCode/bestercode.github.io/blob/master/src/assets/nes.yaml) following yaml's indentation rules.\
Text (e.g. game title or description) that contains the `:` character must be enclosed into parentheses.
