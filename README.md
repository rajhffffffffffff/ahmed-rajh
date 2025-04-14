// Ahmed Rajh - Social Networking Site (Full with Video/Audio Calls) // Java-based backend using Spring Boot framework

// Main Application package com.ahmedrajh;

import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication public class AhmedRajhApplication { public static void main(String[] args) { SpringApplication.run(AhmedRajhApplication.class, args); } }

// User Entity package com.ahmedrajh.model;

import jakarta.persistence.*;

@Entity @Table(name = "users") public class User { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Long id;

private String username;
private String password;
private String email;

// Getters and Setters

}

// User Repository package com.ahmedrajh.repository;

import com.ahmedrajh.model.User; import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> { User findByUsername(String username); }

// User Controller for Login package com.ahmedrajh.controller;

import com.ahmedrajh.model.User; import com.ahmedrajh.repository.UserRepository; import org.springframework.beans.factory.annotation.Autowired; import org.springframework.web.bind.annotation.*;

@RestController @RequestMapping("/api/user") public class UserController {

@Autowired
private UserRepository userRepository;

@PostMapping("/login")
public String login(@RequestBody User user) {
    User existing = userRepository.findByUsername(user.getUsername());
    if (existing != null && existing.getPassword().equals(user.getPassword())) {
        return "Login successful";
    } else {
        return "Invalid credentials";
    }
}

}

// WebSocket Configuration package com.ahmedrajh.config;

import org.springframework.context.annotation.Configuration; import org.springframework.web.socket.config.annotation.*;

@Configuration @EnableWebSocket public class WebSocketConfig implements WebSocketConfigurer { @Override public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) { registry.addHandler(new SignalingSocketHandler(), "/ws").setAllowedOrigins("*"); } }

// WebSocket Signaling Handler package com.ahmedrajh.config;

import org.springframework.web.socket.; import org.springframework.web.socket.handler.TextWebSocketHandler; import java.util.;

public class SignalingSocketHandler extends TextWebSocketHandler { private static final Map<String, WebSocketSession> sessions = new HashMap<>();

@Override
public void afterConnectionEstablished(WebSocketSession session) {
    String userId = session.getId();
    sessions.put(userId, session);
}

@Override
public void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    for (WebSocketSession s : sessions.values()) {
        if (!s.getId().equals(session.getId())) {
            s.sendMessage(message);
        }
    }
}

@Override
public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
    sessions.remove(session.getId());
}

}

// application.properties spring.datasource.url=jdbc:mysql://localhost:3306/ahmedrajh_db spring.datasource.username=root spring.datasource.password=your_password spring.jpa.hibernate.ddl-auto=update

// Frontend HTML + JavaScript for WebRTC (Put in src/main/resources/static/index.html) /*

<!DOCTYPE html><html>
<head>
    <title>Ahmed Rajh Video Call</title>
</head>
<body>
    <h2>Video Chat</h2>
    <video id="localVideo" autoplay muted></video>
    <video id="remoteVideo" autoplay></video>
    <script>
        const localVideo = document.getElementById('localVideo');
        const remoteVideo = document.getElementById('remoteVideo');
        const peer = new RTCPeerConnection();navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then(stream => {
        localVideo.srcObject = stream;
        stream.getTracks().forEach(track => peer.addTrack(track, stream));
    });

    const socket = new WebSocket('ws://localhost:8080/ws');
    socket.onmessage = async (message) => {
        const data = JSON.parse(message.data);
        if (data.offer) {
            await peer.setRemoteDescription(new RTCSessionDescription(data.offer));
            const answer = await peer.createAnswer();
            await peer.setLocalDescription(answer);
            socket.send(JSON.stringify({ answer }));
        } else if (data.answer) {
            await peer.setRemoteDescription(new RTCSessionDescription(data.answer));
        } else if (data.candidate) {
            await peer.addIceCandidate(new RTCIceCandidate(data.candidate));
        }
    };

    peer.onicecandidate = event => {
        if (event.candidate) {
            socket.send(JSON.stringify({ candidate: event.candidate }));
        }
    };

    peer.ontrack = event => {
        remoteVideo.srcObject = event.streams[0];
    };

    async function startCall() {
        const offer = await peer.createOffer();
        await peer.setLocalDescription(offer);
        socket.send(JSON.stringify({ offer }));
    }

    // Start call on page load (for demo only)
    setTimeout(startCall, 1000);
</script>

</body>
</html>
*/
