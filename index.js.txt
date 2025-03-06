const express = require('express');
const multer = require('multer');
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const app = express();
const upload = multer({ dest: 'uploads/' });

app.post('/combine-videos', upload.fields([
  { name: 'twitterVideo', maxCount: 1 },
  { name: 'mockupVideo', maxCount: 1 }
]), (req, res) => {
  const twitterVideoPath = req.files.twitterVideo[0].path;
  const mockupVideoPath = req.files.mockupVideo[0].path;
  const outputPath = path.join('uploads', `output-${Date.now()}.mp4`);

  const command = `ffmpeg -i ${twitterVideoPath} -i ${mockupVideoPath} -filter_complex "[0:v]scale=iw*0.8:ih*0.8[scaled]; [1:v][scaled]overlay=(W-w)/2:(H-h)/2:shortest=1" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -pix_fmt yuv420p -movflags +faststart ${outputPath}`;

  exec(command, (error, stdout, stderr) => {
    if (error) {
      console.error(`Erreur d'exécution: ${error}`);
      return res.status(500).send('Erreur lors du traitement de la vidéo');
    }
    
    res.download(outputPath, 'combined_video.mp4', (err) => {
      if (err) console.error(err);
      
      // Nettoyage des fichiers
      fs.unlinkSync(twitterVideoPath);
      fs.unlinkSync(mockupVideoPath);
      fs.unlinkSync(outputPath);
    });
  });
});

app.listen(process.env.PORT || 3000);
