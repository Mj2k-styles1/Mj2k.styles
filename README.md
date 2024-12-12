#include <iostream>
#include <stack>
#include <string>
#include <cctype>
#include <stdexcept>

using namespace std;

// *Partie 1 : Conversion Base 26*
// Convertir un nombre en base 26 (0-9, A-P) en décimal
int base26ToDecimal(const string &str) {
    int result = 0;
    for (char c : str) {
        if (isdigit(c)) {
            result = result * 26 + (c - '0');
        } else if (c >= 'A' && c <= 'P') {
            result = result * 26 + (c - 'A' + 10);
        } else {
            throw invalid_argument("Invalid base-26 character.");
        }
    }
    return result;
}

// Convertir un nombre décimal en base 26 (0-9, A-P)
string decimalToBase26(int value) {
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

// Vérifier si les parenthèses sont équilibrées
bool areParenthesesBalanced(const string &expr) {
    int balance = 0;
    for (char c : expr) {
        if (c == '(') balance++;
        else if (c == ')') balance--;
        if (balance < 0) return false; // Trop de parenthèses fermées
    }
    return balance == 0; // Vérifie si toutes les parenthèses sont fermées
}

// Fonction pour appliquer une opération arithmétique
int applyOperation(int left, int right, char op) {
    switch (op) {
        case '+': return left + right;
        case '-': return left - right;
        case '*': return left * right;
        case '/':
            if (right == 0) throw runtime_error("Division by zero.");
            return left / right;
        default: throw invalid_argument("Invalid operator.");
    }
}

// Fonction pour déterminer la priorité des opérateurs
int getPrecedence(char op) {
    if (op == '+' || op == '-') return 1; // Faible priorité
    if (op == '*' || op == '/') return 2; // Haute priorité
    return 0; // Parenthèses ou opérateur inconnu
}

// *Partie 2 : Évaluation avec Priorité des Opérations*
int evaluateExpression(const string &expr) {
    stack<int> values;       // Pile pour les valeurs
    stack<char> operators;   // Pile pour les opérateurs

    auto applyTopOperation = [&]() {
        if (values.size() < 2 || operators.empty()) {
            throw invalid_argument("Invalid expression.");
        }
        int right = values.top(); values.pop();
        int left = values.top(); values.pop();
        char op = operators.top(); operators.pop();
        values.push(applyOperation(left, right, op));
    };

    for (size_t i = 0; i < expr.size(); ++i) {
        char c = expr[i];

        if (isspace(c)) {
            // Ignorer les espaces
            continue;
        } else if (isdigit(c) || isalpha(c)) {
            // Lire un nombre en base 26
            string base26;
            while (i < expr.size() && (isdigit(expr[i]) || isalpha(expr[i]))) {
                base26 += expr[i];
                ++i;
            }
            --i; // Revenir d'un caractère, car la boucle avance trop
            values.push(base26ToDecimal(base26));
        } else if (c == '(') {
            operators.push(c);
        } else if (c == ')') {
            // Appliquer toutes les opérations jusqu'à la parenthèse ouvrante
            while (!operators.empty() && operators.top() != '(') {
                applyTopOperation();
            }
            if (!operators.empty()) operators.pop(); // Retirer '('
        } else if (c == '+' || c == '-' || c == '*' || c == '/') {
            // Appliquer les opérations de priorité supérieure ou égale
            while (!operators.empty() && getPrecedence(operators.top()) >= getPrecedence(c)) {
                applyTopOperation();
            }
            operators.push(c);
        } else {
            throw invalid_argument("Invalid character in expression.");
        }
    }

    // Appliquer toutes les opérations restantes
    while (!operators.empty()) {
        applyTopOperation();
    }

    if (values.size() != 1) {
        throw invalid_argument("Invalid expression.");
    }
    return values.top();
}

// *Partie 3 : Programme Principal*
int main() {
    cout << "Calculatrice en Base 26 (0-9, A-P). Entrez 'exit' pour quitter.\n";

    while (true) {
        string input;
        cout << "Entrez une expression: ";
        getline(cin, input);

        if (input == "exit") break;

        try {
            // Vérification des parenthèses
            if (!areParenthesesBalanced(input)) {
                cout << "Erreur: Parenthèses non équilibrées." << endl;
                continue;
            }

            // Évaluation de l'expression
            int result = evaluateExpression(input);

            // Affichage des résultats
            cout << "Résultat (Décimal): " << result << endl;
            cout << "Résultat (Base 26): " << decimalToBase26(result) << endl;

        } catch (const exception &e) {
            cout << "Erreur: " << e.what() << endl;
        }
    }

    return 0;
}
