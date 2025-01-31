#include <iostream>
#include <stack>
#include <string>
#include <cctype>
#include <stdexcept>

using namespace std;

// --- Conversion Base 26 ↔ Base 10 ---
int base26ToBase10(const string &str) {
    int result = 0;
    for (char c : str) {
        if (isdigit(c)) {
            result = result * 10 + (c - '0');
        } else if (c >= 'A' && c <= 'P') {
            result = result * 10 + (c - 'A' + 10);
        } else {
            throw invalid_argument("Caractère invalide en base 26.");
        }
    }
    return result;
}

string base10ToBase26(int value) {
    if (value == 0) return "0";

    string result;
    while (value > 0) {
        int remainder = value % 26;
        if (remainder < 10) {
            result = char('0' + remainder) + result;
        } else {
            result = char('A' + (remainder - 10)) + result;
        }
        value /= 26;
    }
    return result;
}

// --- Vérification des parenthèses ---
bool areParenthesesBalanced(const string &expr) {
    int balance = 0;
    for (char c : expr) {
        if (c == '(') balance++;
        else if (c == ')') balance--;
        if (balance < 0) return false;
    }
    return balance == 0;
}

// --- Application des opérations ---
int applyOperation(int left, int right, char op) {
    switch (op) {
        case '+': return left + right;
        case '-': return left - right;
        case '*': return left * right;
        case '/':
            if (right == 0) throw runtime_error("Division par zéro.");
            return left / right;
        default: throw invalid_argument("Opérateur invalide.");
    }
}

// --- Évaluation de l'expression avec gestion des priorités ---
int evaluateRecursive(const string &expr, size_t &pos) {
    stack<int> values;
    stack<char> operators;

    auto precedence = [](char op) {
        return (op == '*' || op == '/') ? 2 : 1;
    };

    auto applyTopOperation = [&]() {
        if (values.size() < 2 || operators.empty()) {
            throw invalid_argument("Expression invalide.");
        }
        int right = values.top(); values.pop();
        int left = values.top(); values.pop();
        char op = operators.top(); operators.pop();
        values.push(applyOperation(left, right, op));
    };

    while (pos < expr.size()) {
        char c = expr[pos];

        if (isdigit(c) || isalpha(c)) {  // Lecture d'un nombre en base 26
            string base26;
            while (pos < expr.size() && (isdigit(expr[pos]) || isalpha(expr[pos]))) {
                base26 += expr[pos++];
            }
            values.push(base26ToBase10(base26));
        } else if (c == '(') {
            pos++;
            values.push(evaluateRecursive(expr, pos));
        } else if (c == ')') {
            while (!operators.empty()) applyTopOperation();
            pos++;
            break;
        } else if (c == '+' || c == '-' || c == '*' || c == '/') {
            while (!operators.empty() && precedence(operators.top()) >= precedence(c)) {
                applyTopOperation();
            }
            operators.push(c);
            pos++;
        } else if (isspace(c)) {
            pos++;
        } else {
            throw invalid_argument("Caractère invalide dans l'expression.");
        }
    }

    while (!operators.empty()) applyTopOperation();

    if (values.size() != 1) throw invalid_argument("Expression invalide.");
    return values.top();
}

// --- Fonction principale pour évaluer une expression ---
int evaluate(const string &expr) {
    size_t pos = 0;
    return evaluateRecursive(expr, pos);
}

// --- Programme Principal ---
int main() {
    cout << "Calculatrice en Base 26 (0-9, A-P). Entrez 'exit' pour quitter.\n";

    while (true) {
        string input;
        cout << "Entrez une expression: ";
        getline(cin, input);

        if (input == "exit") break;

        try {
            if (!areParenthesesBalanced(input)) {
                cout << "Erreur: Parenthèses non équilibrées." << endl;
                continue;
            }

            int result = evaluate(input);

            cout << "Résultat (Base 10): " << result << endl;
            cout << "Résultat (Base 26): " << base10ToBase26(result) << endl;

        } catch (const exception &e) {
            cout << "Erreur: " << e.what() << endl;
        }
    }

    return 0;
}
