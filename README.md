#include <iostream>
#include <fstream>
#include <stack>
#include <string>
#include <cctype>
#include <stdexcept>
#include <sstream>
#include <map>
#include <climits>

using namespace std;

long long base26ToBase10(const string &str) {
    long long result = 0;
    for (char c : str) {
        if (isdigit(c)) {
            result = result * 26 + (c - '0');
        } else if (c >= 'A' && c <= 'P') {
            result = result * 26 + (c - 'A' + 10);
        } else {
            throw invalid_argument("Caractère invalide en base 26.");
        }
        if (result > LLONG_MAX / 26) {
            throw runtime_error("Débordement de capacité, trop grand nombre.");
        }
    }
    return result;
}

string base10ToBase26(long long value) {
    if (value == 0) return "0";

    string result;
    while (value > 0) {
        long long remainder = value % 26;
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

long long factorial(long long n) {
    if (n < 0) throw invalid_argument("Factoriel d'un nombre négatif.");
    if (n > 20) throw runtime_error("Débordement de capacité avec le factoriel.");
    long long result = 1;
    for (int i = 1; i <= n; ++i) {
        if (result > LLONG_MAX / i) {
            throw runtime_error("Débordement de capacité avec le factoriel.");
        }
        result *= i;
    }
    return result;
}

long long applyOperation(long long left, long long right, char op) {
    switch (op) {
        case '+':
            if ((right > 0 && left > LLONG_MAX - right) || (right < 0 && left < LLONG_MIN - right)) {
                throw runtime_error("Débordement de capacité avec l'addition.");
            }
            return left + right;
        case '-':
            if ((right < 0 && left > LLONG_MAX + right) || (right > 0 && left < LLONG_MIN + right)) {
                throw runtime_error("Débordement de capacité avec la soustraction.");
            }
            return left - right;
        case '*':
            if (left != 0 && right > LLONG_MAX / left) {
                throw runtime_error("Débordement de capacité avec la multiplication.");
            }
            return left * right;
        case '/':
            if (right == 0) throw runtime_error("Division par zéro.");
            return left / right;
        default: throw invalid_argument("Opérateur invalide.");
    }
}

long long evaluate(const string &expr, map<string, long long> &variables) {
    stack<long long> values;
    stack<char> operators;
    stringstream ss(expr);
    string token;

    auto precedence = [](char op) {
        if (op == '*' || op == '/') return 2;
        if (op == '+' || op == '-') return 1;
        return 0;
    };

    auto applyTopOperation = [&]() {
        if (values.size() < 2 || operators.empty()) {
            throw invalid_argument("Expression invalide.");
        }
        long long right = values.top(); values.pop();
        long long left = values.top(); values.pop();
        char op = operators.top(); operators.pop();
        values.push(applyOperation(left, right, op));
    };

    while (ss >> token) {
        if (isdigit(token[0]) || (token[0] >= 'A' && token[0] <= 'P')) {
            values.push(base26ToBase10(token));
        } else if (token[0] == '(') {
            operators.push('(');
        } else if (token[0] == ')') {
            while (!operators.empty() && operators.top() != '(') applyTopOperation();
            if (!operators.empty()) operators.pop();
        } else if (token == "!") {
            long long val = values.top();
            values.pop();
            values.push(factorial(val));
        } else if (token == "+" || token == "-" || token == "*" || token == "/") {
            while (!operators.empty() && precedence(operators.top()) >= precedence(token[0])) applyTopOperation();
            operators.push(token[0]);
        } else if (isalpha(token[0])) {
            if (variables.find(token) != variables.end()) {
                values.push(variables[token]);
            } else {
                throw invalid_argument("Variable non définie : " + token);
            }
        } else {
            throw invalid_argument("Caractère invalide dans l'expression.");
        }
    }

    while (!operators.empty()) applyTopOperation();

    if (values.size() != 1) throw invalid_argument("Expression invalide.");
    return values.top();
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
    map<string, long long> variables;
    while (getline(inFile, line)) {
        try {
            if (!areParenthesesBalanced(line)) {
                outFile << "Erreur: Parenthèses non équilibrées dans '" << line << "'\n";
                continue;
            }

            size_t pos = line.find('=');
            if (pos != string::npos) {
                string varName = line.substr(0, pos);
                string varValue = line.substr(pos + 1);
                variables[varName] = evaluate(varValue, variables);
                continue;
            }

            long long result = evaluate(line, variables);
            outFile << "Expression: " << line << "\n";
            outFile << "Résultat (Base 10): " << result << "\n";
            outFile << "Résultat (Base 26): " << base10ToBase26(result) << "\n\n";

        } catch (const exception &e) {
            outFile << "Erreur: " << e.what() << " dans '" << line << "'\n\n";
        }
    }

    outFile << "Jean de la Croix et Nahun vous remercient pour votre travail\n";

    inFile.close();
    outFile.close();
}

int main() {
    processFile("calcul.txt", "resultat.txt");
    return 0;
}
