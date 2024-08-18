#include <iostream>
#include <vector>
#include <map>
#include <random>

using namespace std;

class Map {
public:
    int width;
    int height;
    int** grid;

    Map(int w, int h) {
        width = w;
        height = h;
        grid = new int*[width];
        for (int i = 0; i < width; i++) {
            grid[i] = new int[height];
            for (int j = 0; j < height; j++) {
                grid[i][j] = 0;
            }
        }
    }

    ~Map() {
        for (int i = 0; i < width; i++) {
            delete[] grid[i];
        }
        delete[] grid;
    }
};

using MovementRules = map<string, pair<int, int>>;

class Robot {
public:
    int energy;
    int fitness_score;
    MovementRules movement_rules;

    Robot(int e, MovementRules mr) : energy(e), movement_rules(move(mr)), fitness_score(0) {}
};

int run_robot(Robot* robot, Map* map);

using Population = vector<Robot*>;

Population generate_population(int population_size);

void evaluate_population(Population& population, Map* map);

Population select_parents(const Population& population);

Robot* crossover(const Robot* parent1, const Robot* parent2, double mutation_rate);

int main() {
    int population_size = 10;
    int num_generations = 10;
    double mutation_rate = 0.1;

    Map* map = new Map(10, 10);

    Population population = generate_population(population_size);

    for (int generation = 0; generation < num_generations; generation++) {
        evaluate_population(population, map);

        Population parents = select_parents(population);

        Population children;
        for (size_t i = 0; i < parents.size(); i += 2) {
            Robot* child = crossover(parents[i], parents[i + 1], mutation_rate);
            children.push_back(child);
        }

        // Cleanup old generation
        for (Robot* r : population) {
            delete r;
        }

        // Replace old population with children
        population = children;
    }

    // Output the final generation of robots
    for (const Robot* r : population) {
        cout << "Fitness Score: " << r->fitness_score << endl;
        cout << "Movement Rules: ";
        for (auto it = r->movement_rules.begin(); it != r->movement_rules.end(); ++it) {
            const auto& direction = it->first;
            const auto& movement = it->second;
            cout << direction << ": (" << movement.first << ", " << movement.second << ") ";
        }
        cout << endl;
    }

    // Cleanup
    delete map;
    for (Robot* r : population) {
        delete r;
    }

    return 0;
}

int run_robot(Robot* robot, Map* map) {
    // Implement the run_robot function based on your requirements
    // Note: This is a placeholder, and you should replace it with your logic
    return 0;
}

Population generate_population(int population_size) {
    Population population;
    for (int i = 0; i < population_size; i++) {
        MovementRules movement_rules;
        random_device rd;
        mt19937 gen(rd());
        uniform_int_distribution<int> dist(-1, 1);
        movement_rules["up"] = {dist(gen), dist(gen)};
        movement_rules["down"] = {dist(gen), dist(gen)};
        movement_rules["left"] = {dist(gen), dist(gen)};
        movement_rules["right"] = {dist(gen), dist(gen)};
        Robot* r = new Robot(100, movement_rules);
        population.push_back(r);
    }
    return population;
}

void evaluate_population(Population& population, Map* map) {
    for (Robot* r : population) {
        int fitness_score = run_robot(r, map);
        r->fitness_score = fitness_score;
    }
}

Population select_parents(const Population& population) {
    Population parents;
    int total_fitness = 0;
    for (const Robot* r : population) {
        total_fitness += r->fitness_score;
    }
    for (const Robot* r : population) {
        double fitness_probability = (double)r->fitness_score / (double)total_fitness;
        random_device rd;
        mt19937 gen(rd());
        uniform_real_distribution<double> dist(0.0, 1.0);
        if (dist(gen) < fitness_probability) {
            parents.push_back(const_cast<Robot*>(r)); // Avoid const correctness issue
        }
    }
    return parents;
}

Robot* crossover(const Robot* parent1, const Robot* parent2, double mutation_rate) {
    MovementRules movement_rules;

    // Use the same random number generator for mutation
    random_device rd;
    mt19937 gen(rd());
    uniform_real_distribution<double> dist(0.0, 1.0);

    // Crossover
    for (auto it = parent1->movement_rules.begin(); it != parent1->movement_rules.end(); ++it) {
        const auto& direction = it->first;
        const auto& movement = (rand() % 2 == 0) ? it->second : parent2->movement_rules.at(direction);
        movement_rules[direction] = movement;
    }

    // Mutation
    for (auto it = movement_rules.begin(); it != movement_rules.end(); ++it) {
        if (dist(gen) < mutation_rate) {
            it->second = {rand() % 3 - 1, rand() % 3 - 1};
        }
    }

    return new Robot(100, movement_rules);
}
