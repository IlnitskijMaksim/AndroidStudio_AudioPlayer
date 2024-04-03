# Створення власного додатку
Для власного додатку було обрано створити аудіо плеєр, який бере пісні з папки.(src/main/assets/Music)

#### Клас для кнопки відтворення та паузи
```java
        playBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mediaPlayer != null && mediaPlayer.isPlaying()) {
                    mediaPlayer.pause();
                    playBtn.setBackgroundResource(android.R.drawable.ic_media_play);
                } else {
                    if (mediaPlayer != null) {
                        mediaPlayer.start();
                        playBtn.setBackgroundResource(android.R.drawable.ic_media_pause);
                    } else {
                        playSong(String.valueOf(new File(songFiles.get(currentPosition))));
                    }
                }
            }
        });
```

#### Клас для кнопки перемикання на наступну та попередню пісню
```java
        previousBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (currentPosition > 0) {
                    currentPosition--;
                } else {
                    currentPosition = songFiles.size() - 1;
                }
                stopAndPlayNewSong();
            }
        });

        nextBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (currentPosition < songFiles.size() - 1) {
                    currentPosition++;
                } else {
                    currentPosition = 0;
                }
                stopAndPlayNewSong();
            }
        });
    }
```

#### Клас, який перевіряє чи була вже ввімкнута якась пісня. Якщо ввімкнута, вимкнути її та увімкнути обрану
```java
    private void stopAndPlayNewSong() {
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = null;
        }
        playSong(songFiles.get(currentPosition));
    }
```

#### Класи, які зчитують інформацію про виконавця, назву пісні та беруть обкладинку пісні
```java
 private void setAlbumArt(File file) {
        MediaMetadataRetriever retriever = new MediaMetadataRetriever();
        retriever.setDataSource(file.getAbsolutePath());
        byte[] albumArtBytes = retriever.getEmbeddedPicture();
        if (albumArtBytes != null) {
            albumArt.setImageBitmap(BitmapFactory.decodeByteArray(albumArtBytes, 0, albumArtBytes.length));
        } else {
            albumArt.setImageResource(android.R.drawable.ic_media_play);
        }
    }

    private String getSongTitleFromFile(String fileName) {
        return fileName.replace(".mp3", "");
    }

    private String getArtistFromFile(String fileName) {
        try {
            AssetManager assetManager = getAssets();
            InputStream inputStream = assetManager.open("Music/" + fileName);
            File tempFile = File.createTempFile("temp", null, getCacheDir());
            FileOutputStream outputStream = new FileOutputStream(tempFile);

            byte[] buffer = new byte[1024];
            int length;
            while ((length = inputStream.read(buffer)) > 0) {
                outputStream.write(buffer, 0, length);
            }

            inputStream.close();
            outputStream.close();

            MediaMetadataRetriever retriever = new MediaMetadataRetriever();
            retriever.setDataSource(tempFile.getAbsolutePath());

            String artist = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ARTIST);

            tempFile.delete();

            return artist != null ? artist : "Невідомий виконавець";
        } catch (IOException e) {
            e.printStackTrace();
            return "Невідомий виконавець";
        }
    }
```

#### Клас, який програє пісню та змінює виконавця, назву пісні та обкладинку у відповідній панелі
```java
    private void playSong(String fileName) {
        try {
            if (mediaPlayer != null) {
                mediaPlayer.release();
                mediaPlayer = null;
            }

            AssetManager assetManager = getAssets();
            InputStream inputStream = assetManager.open("Music/" + fileName);

            File outputFile = new File(getFilesDir(), fileName);
            FileOutputStream outputStream = new FileOutputStream(outputFile);

            byte[] buffer = new byte[1024];
            int length;
            while ((length = inputStream.read(buffer)) > 0) {
                outputStream.write(buffer, 0, length);
            }

            inputStream.close();
            outputStream.close();

            mediaPlayer = new MediaPlayer();
            mediaPlayer.setDataSource(outputFile.getAbsolutePath());
            mediaPlayer.prepare();
            mediaPlayer.start();

            String artist = getArtistFromFile(fileName);

            String songName = fileName.replace(".mp3", "");

            String songTitleAndArtist =  artist + " - " + songName;

            songTitleControlPanel.setText(songTitleAndArtist);

            setAlbumArt(outputFile);

            if (mediaPlayer != null) {
                seekBar.setMax(mediaPlayer.getDuration());
            }
            updateSeekBarProgress();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

#### Класи для таймеру та "прогрес-бару", в якому можна перелистувати пісню
```java
    private void updateSeekBarProgress() {
        if (mediaPlayer != null) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    while (mediaPlayer != null && mediaPlayer.isPlaying()) {
                        try {
                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    int currentPosition = mediaPlayer.getCurrentPosition();
                                    int totalDuration = mediaPlayer.getDuration();

                                    String currentDurationStr = millisecondsToTimer(currentPosition);
                                    currentDurationTextView.setText(currentDurationStr);

                                    String totalDurationStr = millisecondsToTimer(totalDuration);
                                    totalDurationTextView.setText(totalDurationStr);

                                    seekBar.setProgress(currentPosition);
                                }
                            });
                            Thread.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }).start();
        }
    }

    private String millisecondsToTimer(int milliseconds) {
        String finalTimerString = "";
        String secondsString = "";

        int hours = (milliseconds / (1000 * 60 * 60));
        int minutes = (milliseconds % (1000 * 60 * 60)) / (1000 * 60);
        int seconds = ((milliseconds % (1000 * 60 * 60)) % (1000 * 60) / 1000);

        if (seconds < 10) {
            secondsString = "0" + seconds;
        } else {
            secondsString = "" + seconds;
        }

        finalTimerString = minutes + ":" + secondsString;

        return finalTimerString;
    }
```

## Як працює програма

https://github.com/IlnitskijMaksim/AndroidStudio_AudioPlayer/assets/112692170/7e0c9596-4aa9-4410-bb65-3e1d94541e7f

## Що не так?

При швидкому перемиканні або просто через деякий час додаток вилітає при перемиканні пісні. Скоріш за все це пов'язано або з тимчасовими файлами, або з оновленням UI, але це точно пов'язано з оптимізацією.

https://github.com/IlnitskijMaksim/AndroidStudio_AudioPlayer/assets/112692170/b8158417-de06-481d-8e2b-b3e10f559c28

Я не знаю як це зробити, тому залишив так(

В гітхаб залив менше пісень, тому що проект займав би дуже багато місця.



