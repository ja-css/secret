  
Web scraper module - runs on `node.js`.  
  
https://www.npmjs.com/package/website-scraper  
  
1. Install module using npm  
   `npm install website-scraper`  
  
2. Create script file `index.mjs` with content: (change website to scrape and output directory)  
   ```  
   import scrape from 'website-scraper'; // only as ESM, no CommonJS  
   const options = {  
   urls: ['https://unlimitedcryptocharts.com/'],  
   directory: 'unlimitedcryptocharts'  
   };  
   // with async/await  
   const result = await scrape(options);  
   // with promise  
   scrape(options).then((result) => {});  
   ```  
  
3. run script  
   `node index.mjs`