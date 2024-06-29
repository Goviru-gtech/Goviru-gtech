- 👋 Hi, I’m @Goviru-gtech
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Goviru-gtech/Goviru-gtech is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
// ChatApp.java

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;

import java.util.ArrayList;
import java.util.List;

public class ChatApp extends AppCompatActivity {

    // Firebase Authentication
    private FirebaseAuth mAuth;

    // Firebase Realtime Database
    private FirebaseDatabase database;
    private DatabaseReference messagesRef;

    // UI components
    private EditText messageInput;
    private Button sendButton;
    private ListView chatListView;

    // Chat message list
    private List<Message> messageList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat_app);

        // Initialize Firebase Authentication
        mAuth = FirebaseAuth.getInstance();

        // Initialize Firebase Realtime Database
        database = FirebaseDatabase.getInstance();
        messagesRef = database.getReference("messages");

        // Get UI components
        messageInput = findViewById(R.id.message_input);
        sendButton = findViewById(R.id.send_button);
        chatListView = findViewById(R.id.chat_list_view);

        // Set up send button click listener
        sendButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String messageText = messageInput.getText().toString();
                sendMessage(messageText);
                messageInput.setText("");
            }
        });

        // Set up chat list view adapter
        ChatAdapter adapter = new ChatAdapter(this, messageList);
        chatListView.setAdapter(adapter);

        // Load chat messages from Firebase Realtime Database
        loadChatMessages();
    }

    // Send a message to the Firebase Realtime Database
    private void sendMessage(String messageText) {
        FirebaseUser user = mAuth.getCurrentUser();
        if (user!= null) {
            String username = user.getDisplayName();
            Message message = new Message(messageText, username);
            messagesRef.push().setValue(message);
        } else {
            Toast.makeText(this, "You must be logged in to send a message", Toast.LENGTH_SHORT).show();
        }
    }

    // Load chat messages from Firebase Realtime Database
    private void loadChatMessages() {
        messagesRef.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                messageList.clear();
                for (DataSnapshot messageSnapshot : dataSnapshot.getChildren()) {
                    Message message = messageSnapshot.getValue(Message.class);
                    messageList.add(message);
                }
                chatListView.setAdapter(new ChatAdapter(ChatApp.this, messageList));
            }

            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {
                Toast.makeText(ChatApp.this, "Error loading chat messages", Toast.LENGTH_SHORT).show();
            }
        });
    }

    // Log in a user
    public void logInUser(View view) {
        String email = "john.doe@example.com";
        String password = "password";
        mAuth.signInWithEmailAndPassword(email, password)
               .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            Toast.makeText(ChatApp.this, "Logged in successfully", Toast.LENGTH_SHORT).show();
                        } else {
                            Toast.makeText(ChatApp.this, "Error logging in", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
    }

    // Register a new user
    public void registerUser(View view) {
        String email = "jane.doe@example.com";
        String password = "password";
        mAuth.createUserWithEmailAndPassword(email, password)
               .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            Toast.makeText(ChatApp.this, "User created successfully", Toast.LENGTH_SHORT).show();
                        } else {
                            Toast.makeText(ChatApp.this, "Error creating user", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
    }
}

// Message.java

public class Message {
    private String text;
    private String username;

    public Message(String text, String username) {
        this.text = text;
        this.username = username;
    }

    public String getText() {
        return text;
    }

    public String getUsername() {
        return username;
    }
}

// ChatAdapter.java

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.TextView;

import java.util.List;

public class ChatAdapter extends ArrayAdapter<Message> {
    private Context context;
    private List<Message
