Exp 1

package com.example.exp1;

import android.graphics.Color;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

import android.view.View;

import android.widget.Button;

import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

3 usages

TextView textView;

2 usages

Button button, button2;

4 usages

float fontSize = 24;

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(R.layout.activity_main);

textView = findViewById(R.id.textView);

button = findViewById(R.id.button);

button2 = findViewById(R.id.button2);

button.setOnClickListener(new View.OnClickListener() {

@Override

public void onClick(View v) {

textView.setTextColor(Color.parseColor(colorString: *#66FF66*));

}

});

button2.setOnClickListener(new View.OnClickListener() {

@Override

public void onClick(View v) {

textView.setTextSize(fontSize);

fontSize += 4;

if (fontSize == 40) {

fontSize = 20;

}

}

});

}

}

Exp 3

package com.example.exp3;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

import android.view.View;

import android.content.Context;

import android.graphics.Canvas;

import android.graphics.Color;

import android.graphics.Paint;

public class MainActivity extends AppCompatActivity {

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(new GraphicsView(context: this));

}

private static class GraphicsView extends View {

public GraphicsView(Context context) {

super(context);

}

@Override

protected void onDraw(Canvas canvas) {

super.onDraw(canvas);

Paint paint = new Paint();

paint.setStyle(Paint.Style.STROKE);

paint.setStrokeWidth(5f);

// Draw a circle

paint.setColor(Color.RED);

canvas.drawCircle(cx: 150, су: 200, radius: 100, paint);

// Draw a rectangle

paint.setColor(Color.BLUE);

canvas.drawRect(left: 50, top: 300, right: 250, bottom: 400, paint);

// Draw a square

paint.setColor(Color.GREEN);

canvas.drawRect(left: 300, top: 300, right: 450, bottom: 450, paint);

// Draw a line

paint.setColor(Color.YELLOW);

canvas.drawLine(startX: 50, starty: 500, stopX: 450, stopy: 500, paint);
}
}
}


Exp 4

package com.example.exp4;

import androidx.appcompat.app.AppCompatActivity;

import androidx.core.app.NotificationCompat;

import android.app.NotificationManager;

import android.os.Bundle;

import android.view.View;

import android.widget.Button;

public class MainActivity extends AppCompatActivity {

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(R.layout.activity_main);

Button buttonShowNotification = findViewById(R.id.button);

buttonShowNotification.setOnClickListener(new View.OnClickListener() {

@Override

public void onClick(View v) {

showNotification(title: "Simple Notification", message: "This is a simple notification!");

}

});

}

1 usage

private void showNotification(String title, String message) {

NotificationCompat.Builder builder = new NotificationCompat.Builder(context: this, channelld: "default")

.setSmallIcon(android.R.drawable.ic_dialog_info)

.setContentTitle(title)

.setContentText(message)

.setPriority (NotificationCompat.PRIORITY_DEFAULT);

NotificationManager notificationManager = getSystemService (NotificationManager.class);

notificationManager.notify( id: 0, builder.build());

}

}

Exp 6

package com.example.exp6;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

import android.os.AsyncTask;

import android.widget.Button;

import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

2 usages

private Button startButton;

3 usages

private TextView resultTextView;

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(R.layout.activity_main);

startButton = findViewById(R.id.button);

resultTextView = findViewById(R.id.textView);

startButton.setOnClickListener(v -> new BackgroundTask().execute());

}

1 usage

private class Background Task extends AsyncTask<Void, Void, String> {

7 usages

@Override

protected void onPreExecute() {

resultTextView.setText("Task started..

}

@Override

protected String deInBackground(Void... voids) {

try {

Thread.sleep(millis: 3000);

}

}

catch (InterruptedException e) {

e.printStackTrace();

return "Task completed";

}

@Override

protected void onPostExecute(String result) {

resultTextView.setText(result);

}
}
}

Exp 9

package com.example.exp7;

import android.os.Bundle;

import android.view.View;

import android.widget.Button;

import androidx.appcompat.app.AlertDialog;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

@Override

protected void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);

setContentView(R.layout.activity_main);

Button alertButton = findViewById(R.id.button);

alertButton.setOnClickListener(new View.OnClickListener() { @Override

public void onClick(View v) {

showAlert();

}

});

}

1 usage

private void showAlert() {

AlertDialog.Builder builder = new AlertDialog.Builder(context: this); builder.setTitle("Alert!")

.setMessage("This is a simple alert dialog.")

.setPositiveButton(text: "OK" listener: null)

.show();
}
}



