//LabSheet-5
#include <iostream>
#include <vector>
#include <stack>
#include <queue>
#include <string>

class TextEditor {
private:
    std::vector<char> text;           // Array to store characters
    std::stack<std::pair<std::string, std::string>> undoStack; // Stack for undo
    std::stack<std::pair<std::string, std::string>> redoStack; // Stack for redo
    std::queue<char> clipboard;       // Queue for clipboard management

public:
    void insertText(int position, const std::string& inputText) {
        for (size_t i = 0; i < inputText.size(); ++i) {
            text.insert(text.begin() + position + i, inputText[i]);
        }
        undoStack.push({"insert", inputText + "@" + std::to_string(position)});
        while (!redoStack.empty()) redoStack.pop();  // Clear redo stack after a new operation
    }

  void deleteText(int position, int length) {
        std::string deletedText;
        for (int i = 0; i < length; ++i) {
            deletedText += text[position];
            text.erase(text.begin() + position);
        }
        undoStack.push({"delete", deletedText + "@" + std::to_string(position)});
        while (!redoStack.empty()) redoStack.pop();  // Clear redo stack after a new operation
    }

   void undo() {
        if (undoStack.empty()) {
            std::cout << "Nothing to undo\n";
            return;
        }
        auto lastAction = undoStack.top();
        undoStack.pop();
        std::string action = lastAction.first;
        std::string data = lastAction.second;
        int pos = std::stoi(data.substr(data.find('@') + 1));
        std::string content = data.substr(0, data.find('@'));

  if (action == "insert") {
            for (size_t i = 0; i < content.size(); ++i) {
                text.erase(text.begin() + pos);
            }
            redoStack.push({"insert", content + "@" + std::to_string(pos)});
        } else if (action == "delete") {
            for (size_t i = 0; i < content.size(); ++i) {
                text.insert(text.begin() + pos + i, content[i]);
            }
            redoStack.push({"delete", content + "@" + std::to_string(pos)});
        }
    }

  void redo() {
        if (redoStack.empty()) {
            std::cout << "Nothing to redo\n";
            return;
        }
        auto lastAction = redoStack.top();
        redoStack.pop();
        std::string action = lastAction.first;
        std::string data = lastAction.second;
        int pos = std::stoi(data.substr(data.find('@') + 1));
        std::string content = data.substr(0, data.find('@'));

  if (action == "insert") {
            for (size_t i = 0; i < content.size(); ++i) {
                text.insert(text.begin() + pos + i, content[i]);
            }
            undoStack.push({"insert", content + "@" + std::to_string(pos)});
        } else if (action == "delete") {
            for (size_t i = 0; i < content.size(); ++i) {
                text.erase(text.begin() + pos);
            }
            undoStack.push({"delete", content + "@" + std::to_string(pos)});
        }
    }

  void copy(int position, int length) {
        while (!clipboard.empty()) clipboard.pop(); // Clear clipboard before copying
        for (int i = 0; i < length; ++i) {
            clipboard.push(text[position + i]);
        }
    }

  void paste(int position) {
        std::string clipboardText;
        std::queue<char> tempClipboard = clipboard;
        while (!tempClipboard.empty()) {
            clipboardText += tempClipboard.front();
            tempClipboard.pop();
        }
        for (size_t i = 0; i < clipboardText.size(); ++i) {
            text.insert(text.begin() + position + i, clipboardText[i]);
        }
        undoStack.push({"insert", clipboardText + "@" + std::to_string(position)});
        while (!redoStack.empty()) redoStack.pop();  // Clear redo stack after a new operation
    }

  void displayText() const {
        for (char c : text) std::cout << c;
        std::cout << '\n';
    }
};

int main() {
  TextEditor editor;

  //Test cases
  editor.insertText(0, "Hello");
  editor.displayText();  // Expected Output: "Hello"

  editor.deleteText(0, 2);
  editor.displayText();  // Expected Output: "llo"

  editor.undo();
  editor.displayText();  // Expected Output: "Hello"

  editor.redo();
  editor.displayText();  // Expected Output: "llo"

  editor.copy(0, 2);
  editor.paste(3);
  editor.displayText();  // Expected Output: "lloHe"

  return 0;
}
