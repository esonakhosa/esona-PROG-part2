import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class Message {
    private final String messageID;
    private final String recipient;
    private final String messageText;
    private final int messageNumber;
    private final String messageHash;

    private static int totalMessagesSent = 0;
    private static final List<Message> sentMessages = new ArrayList<>();
    private static final List<Message> storedMessages = new ArrayList<>();

    public Message(String recipient, String messageText, int messageNumber) {
        this.messageID = generateMessageID();
        this.recipient = recipient;
        this.messageText = messageText;
        this.messageNumber = messageNumber;
        this.messageHash = createMessageHash();
    }

    public static void main(String[] args) {
        Message message = new Message(
                "+27123456789",
                "Hello, this is a test message.",
                1);

        System.out.println("Message ID valid: " + message.checkMessageID());
        System.out.println(message.checkRecipientCell());
        System.out.println(message.validateMessage());
        System.out.println(message.SentMessage(1));
        System.out.println(message.printMessages());
        System.out.println("Total messages sent: " + returnTotalMessages());
    }

    private String generateMessageID() {
        long number = 1_000_000_000L
                + (long) (new Random().nextDouble() * 9_000_000_000L);
        return String.valueOf(number);
    }

    public boolean checkMessageID() {
        return messageID != null && messageID.length() <= 10;
    }

    public String checkRecipientCell() {
        if (recipient != null && recipient.matches("\\+27\\d{9}")) {
            return "Cell phone number successfully captured.";
        }
        return "Cell phone number is incorrectly formatted or does not contain the international code. Please correct the number and try again.";
    }

    public String validateMessage() {
        if (messageText == null || messageText.isEmpty()) {
            return "Message cannot be empty.";
        }
        if (messageText.length() > 250) {
            int excess = messageText.length() - 250;
            return "Message exceeds 250 characters by "
                    + excess + ", please reduce the size.";
        }
        return "Message ready to send.";
    }

    public String createMessageHash() {
        String text = messageText == null ? "" : messageText.trim();
        String[] words = text.isEmpty() ? new String[0] : text.split("\\s+");

        String firstWord = words.length > 0
                ? words[0].replaceAll("[^a-zA-Z0-9]", "")
                : "";
        String lastWord = words.length > 0
                ? words[words.length - 1].replaceAll("[^a-zA-Z0-9]", "")
                : "";

        return messageID.substring(0, 2)
                + ":" + messageNumber
                + ":" + (firstWord + lastWord).toUpperCase();
    }

    /* Choices: 1 = Send, 2 = Disregard, 3 = Store. */
    public String SentMessage(int choice) {
        switch (choice) {
            case 1:
                sentMessages.add(this);
                totalMessagesSent++;
                return "Message successfully sent.";
            case 2:
                return "Press 0 to delete the message.";
            case 3:
                try {
                    storeMessage();
                    storedMessages.add(this);
                    return "Message successfully stored.";
                } catch (IOException e) {
                    return "Message could not be stored: " + e.getMessage();
                }
            default:
                return "Invalid option.";
        }
    }

    public String printMessages() {
        StringBuilder output = new StringBuilder();
        for (Message message : sentMessages) {
            output.append("Message ID: ").append(message.messageID)
                    .append("\nMessage Hash: ").append(message.messageHash)
                    .append("\nRecipient: ").append(message.recipient)
                    .append("\nMessage: ").append(message.messageText)
                    .append("\n\n");
        }
        return output.length() == 0 ? "No sent messages yet." : output.toString();
    }

    public static int returnTotalMessages() {
        return totalMessagesSent;
    }

    // Appends this message to messages.json as an object in a JSON array.
    public void storeMessage() throws IOException {
        Path path = Path.of("messages.json");
        String existing = "[]";

        if (Files.exists(path)) {
            existing = Files.readString(path, StandardCharsets.UTF_8).trim();
        }

        String object = "{"
                + "\"messageID\":\"" + escape(messageID) + "\","
                + "\"messageHash\":\"" + escape(messageHash) + "\","
                + "\"recipient\":\"" + escape(recipient) + "\","
                + "\"message\":\"" + escape(messageText) + "\""
                + "}";

        String updated;
        if (existing.isEmpty() || existing.equals("[]")) {
            updated = "[\n" + object + "\n]";
        } else if (existing.endsWith("]")) {
            String contents = existing.substring(0, existing.length() - 1).trim();
            if (!contents.startsWith("[")) {
                throw new IOException("messages.json is not a valid JSON array.");
            }
            updated = contents + ",\n" + object + "\n]";
        } else {
            throw new IOException("messages.json is not a valid JSON array.");
        }

        Files.writeString(path, updated, StandardCharsets.UTF_8,
                StandardOpenOption.CREATE,
                StandardOpenOption.TRUNCATE_EXISTING);
    }

    private String escape(String value) {
        if (value == null) {
            return "";
        }
        return value.replace("\\", "\\\\")
                .replace("\"", "\\\"")
                .replace("\n", "\\n")
                .replace("\r", "\\r")
                .replace("\t", "\\t");
    }

    public String getMessageID() {
        return messageID;
    }

    public String getRecipient() {
        return recipient;
    }

    public String getMessageText() {
        return messageText;
    }

    public String getMessageHash() {
        return messageHash;
    }

    public int getMessageNumber() {
        return messageNumber;
    }
}
