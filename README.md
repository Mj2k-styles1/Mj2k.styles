#include <iostream>
#include <fstream>
#include <stack>
#include <string>
#include <cctype>
#include <stdexcept>

using namespace std;

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

bool areParenthesesBalanced(const string &expr) {
    int balance = 0;
    for (char c : expr) {
        if (c == '(') balance++;
        else if (c == ')') balance--;
        if (balance < 0) return false;
    }
    return balance == 0;
}

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

        if (isdigit(c) || isalpha(c)) { 
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

int evaluate(const string &expr) {
    size_t pos = 0;
    return evaluateRecursive(expr, pos);
}

void processFile(const string &inputFile, const string &outputFile) {
    ifstream inFile(inputFile);
    ofstream outFile(outputFile);

    if (!inFile) {
        cerr << "Erreur : Impossible d'ouvrir le fichier " << inputFile << endl;
        return;
    }
    if (!outFile) {
        cerr << "Erreur : Impossible d'ouvrir le fichier " << outputFile << endl;
        return;
    }

    string line;
    while (getline(inFile, line)) {
        try {
            if (!areParenthesesBalanced(line)) {
                outFile << "Erreur: Parenthèses non équilibrées dans '" << line << "'\n";
                continue;
            }

            int result = evaluate(line);
            outFile << "Expression: " << line << "\n";
            outFile << "Résultat (Base 10): " << result << "\n";
            outFile << "Résultat (Base 26): " << base10ToBase26(result) << "\n\n";

        } catch (const exception &e) {
            outFile << "Erreur: " << e.what() << " dans '" << line << "'\n\n";
        }
    }

    inFile.close();
    outFile.close();
    cout << "Traitement terminé. Résultats enregistrés dans " << outputFile << endl;
}

int main() {
    string inputFile = "calcul.txt"; 
    string outputFile = "resultats.txt"; 

    processFile(inputFile, outputFile);

    return 0;
}
