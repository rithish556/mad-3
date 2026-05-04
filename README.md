package com.example.musicapp;

import android.media.MediaPlayer;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.widget.Button;
import android.widget.SeekBar;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    Button btnPrevious, btnPlay, btnNext;
    TextView txtSong, txtCurrentTime, txtTotalTime;
    SeekBar seekBar;
    MediaPlayer mediaPlayer;

    int currentSongIndex = 0;
    boolean isPlaying = false;

    Handler handler = new Handler(Looper.getMainLooper());

    int[] songs = {
            R.raw.song1,
            R.raw.song2,
            R.raw.song3
    };

    String[] songNames = {
            "Song 1",
            "Song 2",
            "Song 3"
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnPrevious = findViewById(R.id.btnPrevious);
        btnPlay = findViewById(R.id.btnPlay);
        btnNext = findViewById(R.id.btnNext);
        txtSong = findViewById(R.id.txtSong);
        txtCurrentTime = findViewById(R.id.txtCurrentTime);
        txtTotalTime = findViewById(R.id.txtTotalTime);
        seekBar = findViewById(R.id.seekBar);

        mediaPlayer = MediaPlayer.create(this, songs[currentSongIndex]);
        txtSong.setText(songNames[currentSongIndex]);

        txtTotalTime.setText(formatTime(mediaPlayer.getDuration()));
        seekBar.setMax(mediaPlayer.getDuration());

        btnPlay.setOnClickListener(v -> {
            if (isPlaying) {
                mediaPlayer.pause();
                handler.removeCallbacksAndMessages(null);
                btnPlay.setText(R.string.play_button);
            } else {
                mediaPlayer.start();
                btnPlay.setText(R.string.pause_button);
                updateSeekBar();
            }
            isPlaying = !isPlaying;
        });

        btnNext.setOnClickListener(v -> changeSong(true));
        btnPrevious.setOnClickListener(v -> changeSong(false));

        seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if (fromUser) {
                    mediaPlayer.seekTo(progress);
                    txtCurrentTime.setText(formatTime(progress));
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) { }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) { }
        });
    }

    private void changeSong(boolean next) {
        if (mediaPlayer != null) {
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = null;
        }

        if (next) {
            currentSongIndex++;
            if (currentSongIndex >= songs.length) currentSongIndex = 0;
        } else {
            currentSongIndex--;
            if (currentSongIndex < 0) currentSongIndex = songs.length - 1;
        }

        mediaPlayer = MediaPlayer.create(this, songs[currentSongIndex]);
        if (mediaPlayer == null) {
            return;
        }
        txtSong.setText(songNames[currentSongIndex]);
        txtTotalTime.setText(formatTime(mediaPlayer.getDuration()));
        seekBar.setMax(mediaPlayer.getDuration());

        mediaPlayer.start();
        btnPlay.setText(R.string.pause_button);
        isPlaying = true;

        updateSeekBar();
    }

    private void updateSeekBar() {
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            seekBar.setProgress(mediaPlayer.getCurrentPosition());
            txtCurrentTime.setText(formatTime(mediaPlayer.getCurrentPosition()));
            handler.postDelayed(this::updateSeekBar, 500);
        }
    }

    private String formatTime(int milliseconds) {
        int minutes = milliseconds / 1000 / 60;
        int seconds = (milliseconds / 1000) % 60;
        return String.format(Locale.getDefault(), "%d:%02d", minutes, seconds);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mediaPlayer != null) {
            mediaPlayer.release();
            mediaPlayer = null;
        }
        handler.removeCallbacksAndMessages(null);
    }
}
