# Как работает искусственный интеллект в играх

## 🤖 Что такое игровой ИИ

### Основные задачи игрового ИИ

**Игровой искусственный интеллект** — это программные системы, которые создают иллюзию разумного поведения компьютерных персонажей и противников.

**Основные цели:**
1. **Создать вызов** — игра должна быть интересной и сложной
2. **Имитировать реальное поведение** — персонажи должны действовать правдоподобно
3. **Адаптироваться** — подстраиваться под уровень игрока
4. **Развлекать** — делать игру увлекательной

### Отличия от "настоящего" ИИ

**Игровой ИИ vs Научный ИИ:**

| Игровой ИИ | Научный ИИ |
|------------|------------|
| Цель — развлечение | Цель — решение проблем |
| Может "поддаваться" | Всегда стремится к оптимуму |
| Предсказуемое поведение | Непредсказуемые результаты |
| Ограниченные ресурсы | Максимум вычислительной мощности |
| Должен быть понятным игроку | Может быть "чёрным ящиком" |

---

## 🎯 Типы игрового ИИ

### 1. Конечные автоматы (Finite State Machines)

**Принцип:** ИИ находится в одном из заранее определённых состояний и переключается между ними.

```
Состояния NPC охранника:
[Патрулирование] → [Подозрение] → [Преследование] → [Атака]
       ↑                                                  ↓
       ←←←←←←←←←←← [Возвращение] ←←←←←←←←←←←←←←←←←←←←←←←←←
```

**Пример FSM:**
```python
class GuardAI:
    def __init__(self):
        self.state = "patrol"
        self.player_distance = 100

    def update(self):
        if self.state == "patrol":
            if self.player_distance < 10:
                self.state = "suspicious"

        elif self.state == "suspicious":
            if self.player_distance < 5:
                self.state = "chase"
            elif self.player_distance > 15:
                self.state = "patrol"

        elif self.state == "chase":
            if self.player_distance > 20:
                self.state = "return"
            elif self.player_distance < 2:
                self.state = "attack"
```

### 2. Деревья поведений (Behavior Trees)

**Принцип:** Иерархическая структура решений с условиями и действиями.

```
                    Root
                     |
                 Selector
               /      |      \
         Sequence  Sequence  Action
         /     \    /     \    (Idle)
    Condition Action Condition Action
   (EnemyNear)(Attack)(LowHealth)(Heal)
```

### 3. Алгоритмы планирования

**Goal-Oriented Action Planning (GOAP):**
- ИИ имеет цель
- Планирует последовательность действий для достижения цели
- Адаптируется к изменениям в мире

### 4. Машинное обучение

**Нейронные сети, генетические алгоритмы, обучение с подкреплением**

---

## 🎮 Крестики-нолики: пример с ботом

### Простейший ИИ: случайные ходы

```python
import random

class RandomBot:
    def make_move(self, board):
        """Делает случайный ход"""
        empty_cells = []

        # Находим пустые клетки
        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    empty_cells.append((i, j))

        # Выбираем случайную пустую клетку
        if empty_cells:
            return random.choice(empty_cells)
        return None
```

### Улучшенный ИИ: правила

```python
class RuleBasedBot:
    def make_move(self, board, player='O'):
        """
        Бот с простыми правилами:
        1. Выиграть, если можешь
        2. Заблокировать противника, если он может выиграть
        3. Занять центр
        4. Занять угол
        5. Случайный ход
        """

        # 1. Попытка выиграть
        move = self.find_winning_move(board, player)
        if move:
            return move

        # 2. Блокировка противника
        opponent = 'X' if player == 'O' else 'O'
        move = self.find_winning_move(board, opponent)
        if move:
            return move

        # 3. Занять центр
        if board[1][1] == ' ':
            return (1, 1)

        # 4. Занять угол
        corners = [(0,0), (0,2), (2,0), (2,2)]
        for corner in corners:
            if board[corner[0]][corner[1]] == ' ':
                return corner

        # 5. Любая пустая клетка
        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    return (i, j)

        return None

    def find_winning_move(self, board, player):
        """Находит ход для победы"""
        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    # Пробуем ход
                    board[i][j] = player
                    if self.check_winner(board) == player:
                        board[i][j] = ' '  # Отменяем ход
                        return (i, j)
                    board[i][j] = ' '  # Отменяем ход
        return None
```

### Продвинутый ИИ: алгоритм Minimax

**Принцип:** ИИ просчитывает все возможные ходы на несколько шагов вперёд и выбирает оптимальный.

```python
class MinimaxBot:
    def __init__(self, difficulty='hard'):
        self.difficulty = difficulty
        self.max_depth = {'easy': 1, 'medium': 3, 'hard': 9}[difficulty]

    def make_move(self, board, player='O'):
        """Использует алгоритм Minimax"""
        _, best_move = self.minimax(board, self.max_depth, True, player)
        return best_move

    def minimax(self, board, depth, is_maximizing, player):
        """
        Алгоритм Minimax:
        - Maximizing player пытается максимизировать счёт
        - Minimizing player пытается минимизировать счёт
        """

        # Проверяем конец игры
        winner = self.check_winner(board)
        if winner == player:
            return 10 - depth, None  # Быстрая победа лучше медленной
        elif winner and winner != player:
            return depth - 10, None  # Быстрое поражение хуже медленного
        elif self.is_board_full(board):
            return 0, None  # Ничья

        # Если достигли максимальной глубины
        if depth == 0:
            return 0, None

        best_move = None

        if is_maximizing:
            max_eval = float('-inf')

            for i in range(3):
                for j in range(3):
                    if board[i][j] == ' ':
                        # Делаем ход
                        board[i][j] = player

                        # Рекурсивно вычисляем оценку
                        eval_score, _ = self.minimax(board, depth - 1, False, player)

                        # Отменяем ход
                        board[i][j] = ' '

                        # Обновляем лучший ход
                        if eval_score > max_eval:
                            max_eval = eval_score
                            best_move = (i, j)

            return max_eval, best_move

        else:  # Minimizing
            min_eval = float('inf')
            opponent = 'X' if player == 'O' else 'O'

            for i in range(3):
                for j in range(3):
                    if board[i][j] == ' ':
                        # Ход противника
                        board[i][j] = opponent

                        # Рекурсивно вычисляем оценку
                        eval_score, _ = self.minimax(board, depth - 1, True, player)

                        # Отменяем ход
                        board[i][j] = ' '

                        # Обновляем худший для нас ход
                        if eval_score < min_eval:
                            min_eval = eval_score
                            best_move = (i, j)

            return min_eval, best_move

    def check_winner(self, board):
        """Проверяет победителя"""
        lines = []
        for i in range(3):
            lines.append([board[i][j] for j in range(3)])
            lines.append([board[j][i] for j in range(3)])
        lines.append([board[i][i] for i in range(3)])
        lines.append([board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]
        return None

    def is_board_full(self, board):
        """Проверяет, заполнена ли доска"""
        return all(board[i][j] != ' ' for i in range(3) for j in range(3))
```

### Оптимизированный Minimax: Alpha-Beta отсечение

```python
class AlphaBetaBot(MinimaxBot):
    def minimax_alpha_beta(self, board, depth, alpha, beta, is_maximizing, player):
        """
        Alpha-Beta pruning оптимизация:
        - Alpha: лучший результат для максимизирующего игрока
        - Beta: лучший результат для минимизирующего игрока
        - Если alpha >= beta, можно прекратить поиск (отсечение)
        """

        # Проверки конца игры (как в обычном minimax)
        winner = self.check_winner(board)
        if winner == player:
            return 10 - depth, None
        elif winner and winner != player:
            return depth - 10, None
        elif self.is_board_full(board) or depth == 0:
            return 0, None

        best_move = None

        if is_maximizing:
            max_eval = float('-inf')

            for i in range(3):
                for j in range(3):
                    if board[i][j] == ' ':
                        board[i][j] = player

                        eval_score, _ = self.minimax_alpha_beta(
                            board, depth - 1, alpha, beta, False, player
                        )

                        board[i][j] = ' '

                        if eval_score > max_eval:
                            max_eval = eval_score
                            best_move = (i, j)

                        alpha = max(alpha, eval_score)

                        # Alpha-Beta отсечение
                        if beta <= alpha:
                            break  # Прекращаем поиск

            return max_eval, best_move

        else:
            min_eval = float('inf')
            opponent = 'X' if player == 'O' else 'O'

            for i in range(3):
                for j in range(3):
                    if board[i][j] == ' ':
                        board[i][j] = opponent

                        eval_score, _ = self.minimax_alpha_beta(
                            board, depth - 1, alpha, beta, True, player
                        )

                        board[i][j] = ' '

                        if eval_score < min_eval:
                            min_eval = eval_score
                            best_move = (i, j)

                        beta = min(beta, eval_score)

                        # Alpha-Beta отсечение
                        if beta <= alpha:
                            break

            return min_eval, best_move
```

---

## 🧠 Более сложные игры и алгоритмы

### Monte Carlo Tree Search (MCTS)

**Используется в:** Go, покер, общие настольные игры

**Принцип:**
1. **Selection** — выбираем путь в дереве до листа
2. **Expansion** — добавляем новые узлы (возможные ходы)
3. **Simulation** — играем случайную партию до конца
4. **Backpropagation** — обновляем статистику всех узлов на пути

```python
class MCTSNode:
    def __init__(self, state, parent=None):
        self.state = state
        self.parent = parent
        self.children = []
        self.visits = 0
        self.wins = 0
        self.untried_moves = self.get_legal_moves()

    def uct_select_child(self):
        """Upper Confidence Bound для выбора лучшего ребёнка"""
        import math

        return max(
            self.children,
            key=lambda child: (
                child.wins / child.visits +
                math.sqrt(2 * math.log(self.visits) / child.visits)
            )
        )

    def add_child(self, move):
        """Добавляет ребёнка для данного хода"""
        new_state = self.state.make_move(move)
        child = MCTSNode(new_state, self)
        self.untried_moves.remove(move)
        self.children.append(child)
        return child

    def update(self, result):
        """Обновляет статистику узла"""
        self.visits += 1
        self.wins += result

    def backpropagate(self, result):
        """Обновляет статистику до корня дерева"""
        self.update(result)
        if self.parent:
            self.parent.backpropagate(1 - result)  # Инвертируем для противника
```

### Нейронные сети в играх

**AlphaGo/AlphaZero подход:**

```python
import numpy as np
import tensorflow as tf

class GameAI:
    def __init__(self):
        self.model = self.build_network()
        self.mcts = MCTS(self.model)

    def build_network(self):
        """Нейронная сеть с двумя выходами: policy и value"""

        # Входной слой (состояние игры)
        input_layer = tf.keras.layers.Input(shape=(3, 3, 1))

        # Свёрточные слои
        x = tf.keras.layers.Conv2D(32, 3, padding='same', activation='relu')(input_layer)
        x = tf.keras.layers.Conv2D(64, 3, padding='same', activation='relu')(x)

        # Flatten для полносвязных слоев
        x = tf.keras.layers.Flatten()(x)
        x = tf.keras.layers.Dense(128, activation='relu')(x)

        # Policy head (вероятности ходов)
        policy = tf.keras.layers.Dense(9, activation='softmax', name='policy')(x)

        # Value head (оценка позиции)
        value = tf.keras.layers.Dense(1, activation='tanh', name='value')(x)

        model = tf.keras.Model(inputs=input_layer, outputs=[policy, value])

        return model

    def predict(self, state):
        """Предсказывает policy и value для данного состояния"""
        state_tensor = np.array(state).reshape(1, 3, 3, 1)
        policy, value = self.model.predict(state_tensor)
        return policy[0], value[0][0]

    def train(self, training_data):
        """Обучает нейронную сеть"""
        states, policies, values = zip(*training_data)

        states = np.array(states).reshape(-1, 3, 3, 1)
        policies = np.array(policies)
        values = np.array(values)

        self.model.fit(
            states,
            [policies, values],
            batch_size=32,
            epochs=10,
            validation_split=0.2
        )
```

### Генетические алгоритмы

**Эволюция игровых стратегий:**

```python
import random
import numpy as np

class GeneticBot:
    def __init__(self, genes=None):
        # Гены определяют веса для разных стратегий
        if genes is None:
            self.genes = {
                'center_weight': random.random(),
                'corner_weight': random.random(),
                'edge_weight': random.random(),
                'block_weight': random.random(),
                'win_weight': random.random() * 10  # Победа важнее всего
            }
        else:
            self.genes = genes

    def evaluate_move(self, board, move, player):
        """Оценивает ход на основе генов"""
        score = 0
        row, col = move

        # Центр
        if (row, col) == (1, 1):
            score += self.genes['center_weight']

        # Углы
        if (row, col) in [(0,0), (0,2), (2,0), (2,2)]:
            score += self.genes['corner_weight']

        # Края
        if (row, col) in [(0,1), (1,0), (1,2), (2,1)]:
            score += self.genes['edge_weight']

        # Имитируем ход и проверяем результат
        board[row][col] = player

        # Выигрышный ход
        if self.check_winner(board) == player:
            score += self.genes['win_weight']

        # Блокирующий ход
        opponent = 'X' if player == 'O' else 'O'
        board[row][col] = opponent
        if self.check_winner(board) == opponent:
            score += self.genes['block_weight']

        # Отменяем ход
        board[row][col] = ' '

        return score

    def make_move(self, board, player='O'):
        """Выбирает лучший ход на основе оценок"""
        best_score = -1
        best_move = None

        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    score = self.evaluate_move(board, (i, j), player)
                    if score > best_score:
                        best_score = score
                        best_move = (i, j)

        return best_move

    def crossover(self, other):
        """Скрещивание двух ботов"""
        child_genes = {}
        for key in self.genes:
            # Случайно выбираем ген от одного из родителей
            if random.random() < 0.5:
                child_genes[key] = self.genes[key]
            else:
                child_genes[key] = other.genes[key]

        return GeneticBot(child_genes)

    def mutate(self, mutation_rate=0.1):
        """Мутация генов"""
        for key in self.genes:
            if random.random() < mutation_rate:
                self.genes[key] += random.gauss(0, 0.1)  # Гауссовская мутация
                self.genes[key] = max(0, self.genes[key])  # Не даём быть отрицательными

    def check_winner(self, board):
        """Проверяет победителя"""
        lines = []
        for i in range(3):
            lines.append([board[i][j] for j in range(3)])
            lines.append([board[j][i] for j in range(3)])
        lines.append([board[i][i] for i in range(3)])
        lines.append([board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]
        return None

class GeneticEvolution:
    def __init__(self, population_size=50):
        self.population = [GeneticBot() for _ in range(population_size)]
        self.generation = 0

    def evaluate_fitness(self, bot):
        """Оценивает качество бота через игры"""
        wins = 0
        games = 20

        for _ in range(games):
            # Играем против случайного бота
            opponent = RandomBot()
            result = self.play_game(bot, opponent)
            if result == 1:  # Победа
                wins += 1
            elif result == 0:  # Ничья
                wins += 0.5

        return wins / games

    def evolve(self):
        """Один шаг эволюции"""
        # Оценяваем всю популяцию
        fitness_scores = [(bot, self.evaluate_fitness(bot)) for bot in self.population]
        fitness_scores.sort(key=lambda x: x[1], reverse=True)

        # Берём лучших 50%
        elite_count = len(self.population) // 2
        elite = [bot for bot, score in fitness_scores[:elite_count]]

        # Создаём новое поколение
        new_population = elite.copy()

        while len(new_population) < len(self.population):
            # Выбираем двух случайных родителей из элиты
            parent1 = random.choice(elite)
            parent2 = random.choice(elite)

            # Создаём потомка
            child = parent1.crossover(parent2)
            child.mutate()

            new_population.append(child)

        self.population = new_population
        self.generation += 1

        best_fitness = fitness_scores[0][1]
        print(f"Поколение {self.generation}, лучший результат: {best_fitness:.2f}")

        return fitness_scores[0][0]  # Возвращаем лучшего бота

    def play_game(self, bot1, bot2):
        """Играет партию между двумя ботами"""
        board = [[' ' for _ in range(3)] for _ in range(3)]
        current_player = 'X'
        moves = 0

        while moves < 9:
            if current_player == 'X':
                bot = bot1
            else:
                bot = bot2

            move = bot.make_move(board, current_player)
            if move is None:
                break

            board[move[0]][move[1]] = current_player
            moves += 1

            winner = self.check_winner(board)
            if winner:
                if winner == 'X':
                    return 1  # Победа bot1
                else:
                    return -1  # Победа bot2

            current_player = 'O' if current_player == 'X' else 'X'

        return 0  # Ничья

    def check_winner(self, board):
        """Проверяет победителя"""
        lines = []
        for i in range(3):
            lines.append([board[i][j] for j in range(3)])
            lines.append([board[j][i] for j in range(3)])
        lines.append([board[i][i] for i in range(3)])
        lines.append([board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]
        return None
```

---

## 🎯 Специализированные техники игрового ИИ

### Поведенческие системы

#### Flocking (Стайное поведение)

```python
class Boid:
    """Элемент стаи (птица, рыба, юнит)"""
    def __init__(self, x, y):
        self.position = np.array([x, y])
        self.velocity = np.random.rand(2) * 2 - 1
        self.acceleration = np.zeros(2)

    def flock(self, boids):
        """Три правила стайного поведения"""
        sep = self.separate(boids)    # Разделение
        ali = self.align(boids)       # Выравнивание
        coh = self.cohesion(boids)    # Сплочённость

        # Веса важности каждого правила
        sep *= 1.5
        ali *= 1.0
        coh *= 1.0

        self.acceleration = sep + ali + coh

    def separate(self, boids):
        """Избегает столкновений с соседями"""
        desired_separation = 25
        steer = np.zeros(2)
        count = 0

        for boid in boids:
            distance = np.linalg.norm(self.position - boid.position)
            if 0 < distance < desired_separation:
                diff = self.position - boid.position
                diff = diff / distance  # Нормализуем
                steer += diff
                count += 1

        if count > 0:
            steer /= count
            steer = self.normalize(steer)

        return steer

    def align(self, boids):
        """Движется в том же направлении, что и соседи"""
        neighbor_distance = 50
        sum_velocity = np.zeros(2)
        count = 0

        for boid in boids:
            distance = np.linalg.norm(self.position - boid.position)
            if 0 < distance < neighbor_distance:
                sum_velocity += boid.velocity
                count += 1

        if count > 0:
            sum_velocity /= count
            return self.normalize(sum_velocity)

        return np.zeros(2)

    def cohesion(self, boids):
        """Движется к центру соседних элементов"""
        neighbor_distance = 50
        sum_position = np.zeros(2)
        count = 0

        for boid in boids:
            distance = np.linalg.norm(self.position - boid.position)
            if 0 < distance < neighbor_distance:
                sum_position += boid.position
                count += 1

        if count > 0:
            sum_position /= count
            return self.seek(sum_position)

        return np.zeros(2)

    def normalize(self, vector):
        """Нормализует вектор"""
        magnitude = np.linalg.norm(vector)
        if magnitude > 0:
            return vector / magnitude
        return vector

    def seek(self, target):
        """Движется к цели"""
        desired = target - self.position
        desired = self.normalize(desired)
        return desired
```

#### Навигация по навмешу (NavMesh)

```python
class NavMesh:
    """Система навигации для ИИ персонажей"""
    def __init__(self):
        self.nodes = {}
        self.connections = {}

    def add_node(self, node_id, x, y):
        """Добавляет узел навигации"""
        self.nodes[node_id] = (x, y)
        self.connections[node_id] = []

    def connect_nodes(self, node1, node2, cost=None):
        """Соединяет два узла"""
        if cost is None:
            pos1 = self.nodes[node1]
            pos2 = self.nodes[node2]
            cost = ((pos1[0] - pos2[0])**2 + (pos1[1] - pos2[1])**2)**0.5

        self.connections[node1].append((node2, cost))
        self.connections[node2].append((node1, cost))

    def a_star_pathfind(self, start, goal):
        """Алгоритм A* для поиска пути"""
        from heapq import heappush, heappop

        open_set = [(0, start)]
        came_from = {}
        g_score = {node: float('inf') for node in self.nodes}
        g_score[start] = 0
        f_score = {node: float('inf') for node in self.nodes}
        f_score[start] = self.heuristic(start, goal)

        while open_set:
            current = heappop(open_set)[1]

            if current == goal:
                # Восстанавливаем путь
                path = []
                while current in came_from:
                    path.append(current)
                    current = came_from[current]
                path.append(start)
                return path[::-1]

            for neighbor, cost in self.connections[current]:
                tentative_g_score = g_score[current] + cost

                if tentative_g_score < g_score[neighbor]:
                    came_from[neighbor] = current
                    g_score[neighbor] = tentative_g_score
                    f_score[neighbor] = g_score[neighbor] + self.heuristic(neighbor, goal)
                    heappush(open_set, (f_score[neighbor], neighbor))

        return []  # Путь не найден

    def heuristic(self, node1, node2):
        """Эвристическая функция (расстояние по прямой)"""
        pos1 = self.nodes[node1]
        pos2 = self.nodes[node2]
        return ((pos1[0] - pos2[0])**2 + (pos1[1] - pos2[1])**2)**0.5
```

### Адаптивный ИИ

#### Обучение с подкреплением в реальном времени

```python
class QLearningBot:
    """Бот, который учится играть через Q-Learning"""

    def __init__(self, learning_rate=0.1, discount=0.95, exploration=0.1):
        self.q_table = {}
        self.learning_rate = learning_rate
        self.discount = discount
        self.exploration = exploration
        self.last_state = None
        self.last_action = None

    def get_state_key(self, board):
        """Преобразует состояние доски в ключ"""
        return ''.join([''.join(row) for row in board])

    def get_q_value(self, state, action):
        """Получает Q-значение для состояния и действия"""
        key = (state, action)
        return self.q_table.get(key, 0.0)

    def make_move(self, board, player='O'):
        """Выбирает действие по epsilon-greedy стратегии"""
        state_key = self.get_state_key(board)

        # Находим все возможные действия
        possible_moves = []
        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    possible_moves.append((i, j))

        if not possible_moves:
            return None

        # Epsilon-greedy выбор
        if random.random() < self.exploration:
            # Исследование: случайный ход
            action = random.choice(possible_moves)
        else:
            # Эксплуатация: лучший известный ход
            best_action = None
            best_q = float('-inf')

            for action in possible_moves:
                q_value = self.get_q_value(state_key, action)
                if q_value > best_q:
                    best_q = q_value
                    best_action = action

            action = best_action if best_action else random.choice(possible_moves)

        self.last_state = state_key
        self.last_action = action

        return action

    def learn(self, reward, new_board=None):
        """Обновляет Q-таблицу на основе полученного вознаграждения"""
        if self.last_state is None or self.last_action is None:
            return

        current_q = self.get_q_value(self.last_state, self.last_action)

        # Находим максимальное Q-значение для нового состояния
        max_future_q = 0
        if new_board:
            new_state_key = self.get_state_key(new_board)
            possible_moves = []

            for i in range(3):
                for j in range(3):
                    if new_board[i][j] == ' ':
                        possible_moves.append((i, j))

            if possible_moves:
                max_future_q = max([
                    self.get_q_value(new_state_key, move)
                    for move in possible_moves
                ])

        # Обновляем Q-значение по формуле Беллмана
        new_q = current_q + self.learning_rate * (
            reward + self.discount * max_future_q - current_q
        )

        self.q_table[(self.last_state, self.last_action)] = new_q

        # Уменьшаем исследование со временем
        if self.exploration > 0.01:
            self.exploration *= 0.995

        # Находим максимальное Q-значение для нового состояния
        max_future_q = 0
        if new_board:
            new_state_key = self.get_state_key(new_board)
            possible_moves = []

            for i in range(3):
                for j in range(3):
                    if new_board[i][j] == ' ':
                        possible_moves.append((i, j))

            if possible_moves:
                max_future_q = max([
                    self.get_q_value(new_state_key, move)
                    for move in possible_moves
                ])

        # Обновляем Q-значение по формуле Беллмана
        new_q = current_q + self.learning_rate * (
            reward + self.discount * max_future_q - current_q
        )

        self.q_table[(self.last_state, self.last_action)] = new_q

        # Уменьшаем исследование со временем
        if self.exploration > 0.01:
            self.exploration *= 0.995

    def save_knowledge(self, filename):
        """Сохраняет изученные Q-значения"""
        import pickle
        with open(filename, 'wb') as f:
            pickle.dump(self.q_table, f)

    def load_knowledge(self, filename):
        """Загружает изученные Q-значения"""
        import pickle
        try:
            with open(filename, 'rb') as f:
                self.q_table = pickle.load(f)
        except FileNotFoundError:
            print("Файл знаний не найден, начинаю с чистого листа")

class GameTrainer:
    """Тренирует Q-Learning бота"""

    def __init__(self):
        self.bot = QLearningBot()
        self.games_played = 0

    def train(self, episodes=10000):
        """Тренирует бота через самоигру"""
        wins = 0
        draws = 0
        losses = 0

        for episode in range(episodes):
            board = [[' ' for _ in range(3)] for _ in range(3)]
            game_over = False
            move_count = 0

            # Играем партию
            while not game_over and move_count < 9:
                current_player = 'X' if move_count % 2 == 0 else 'O'

                if current_player == 'O':  # Наш бот
                    move = self.bot.make_move(board, 'O')
                    if move:
                        board[move[0]][move[1]] = 'O'
                else:  # Противник (случайный или другой ИИ)
                    move = self.random_move(board)
                    if move:
                        board[move[0]][move[1]] = 'X'

                move_count += 1

                # Проверяем результат игры
                winner = self.check_winner(board)
                if winner:
                    game_over = True

                    # Награждаем бота
                    if winner == 'O':
                        reward = 1.0  # Победа
                        wins += 1
                    else:
                        reward = -1.0  # Поражение
                        losses += 1

                    self.bot.learn(reward, board)

                elif move_count == 9:  # Ничья
                    game_over = True
                    reward = 0.1  # Небольшая награда за ничью
                    draws += 1
                    self.bot.learn(reward, board)
                else:
                    # Промежуточная награда за продолжение игры
                    self.bot.learn(0.01, board)

            # Статистика каждые 1000 игр
            if (episode + 1) % 1000 == 0:
                total = wins + draws + losses
                print(f"Эпизод {episode + 1}:")
                print(f"  Победы: {wins/total*100:.1f}%")
                print(f"  Ничьи: {draws/total*100:.1f}%")
                print(f"  Поражения: {losses/total*100:.1f}%")
                print(f"  Исследование: {self.bot.exploration:.3f}")
                print()

    def random_move(self, board):
        """Случайный ход для противника"""
        moves = []
        for i in range(3):
            for j in range(3):
                if board[i][j] == ' ':
                    moves.append((i, j))

        return random.choice(moves) if moves else None

    def check_winner(self, board):
        """Проверяет победителя"""
        # Строки, столбцы, диагонали
        lines = []

        # Строки и столбцы
        for i in range(3):
            lines.append([board[i][j] for j in range(3)])  # Строка
            lines.append([board[j][i] for j in range(3)])  # Столбец

        # Диагонали
        lines.append([board[i][i] for i in range(3)])
        lines.append([board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]

        return None

### 🎯 Нейроэволюция: эволюция нейросетей

**Комбинация генетических алгоритмов и нейронных сетей:**

```python
import numpy as np

class NeuralNetworkBot:
    """Бот на основе нейросети"""

    def __init__(self, weights=None):
        # Архитектура: 9 входов (поле) -> 18 скрытых -> 9 выходов (ходы)
        self.input_size = 9
        self.hidden_size = 18
        self.output_size = 9

        if weights is None:
            # Случайные веса при создании
            self.w1 = np.random.randn(self.input_size, self.hidden_size) * 0.5
            self.b1 = np.random.randn(self.hidden_size) * 0.5
            self.w2 = np.random.randn(self.hidden_size, self.output_size) * 0.5
            self.b2 = np.random.randn(self.output_size) * 0.5
        else:
            # Задаёмся весы извне
            self.set_weights(weights)

    def set_weights(self, weights):
        """Устанавливает веса сети"""
        start = 0

        # Первый слой
        w1_size = self.input_size * self.hidden_size
        self.w1 = weights[start:start+w1_size].reshape(self.input_size, self.hidden_size)
        start += w1_size

        self.b1 = weights[start:start+self.hidden_size]
        start += self.hidden_size

        # Второй слой
        w2_size = self.hidden_size * self.output_size
        self.w2 = weights[start:start+w2_size].reshape(self.hidden_size, self.output_size)
        start += w2_size

        self.b2 = weights[start:start+self.output_size]

    def get_weights(self):
        """Возвращает все веса как одномерный массив"""
        return np.concatenate([
            self.w1.flatten(),
            self.b1,
            self.w2.flatten(),
            self.b2
        ])

    def sigmoid(self, x):
        """Сигмоида активации"""
        return 1 / (1 + np.exp(-np.clip(x, -500, 500)))

    def forward(self, board_state):
        """Прямой проход через сеть"""
        # Преобразуем доску в числовой вход
        x = np.array([
            1 if cell == 'O' else (-1 if cell == 'X' else 0)
            for row in board_state for cell in row
        ], dtype=np.float32)

        # Первый слой
        hidden = self.sigmoid(np.dot(x, self.w1) + self.b1)

        # Выходной слой
        output = self.sigmoid(np.dot(hidden, self.w2) + self.b2)

        return output

    def make_move(self, board, player='O'):
        """Делает ход на основе предсказания сети"""
        output = self.forward(board)

        # Сортируем ходы по предпочтению сети
        preferences = [(i, output[i]) for i in range(9)]
        preferences.sort(key=lambda x: x[1], reverse=True)

        # Выбираем первый доступный ход
        for move_idx, confidence in preferences:
            row, col = move_idx // 3, move_idx % 3
            if board[row][col] == ' ':
                return (row, col)

        return None

class NeuroEvolution:
    """Эволюция нейросетей для игры в крестики-нолики"""

    def __init__(self, population_size=100):
        self.population_size = population_size
        self.population = [NeuralNetworkBot() for _ in range(population_size)]
        self.generation = 0

        # Размер генома (всех весов)
        example_bot = NeuralNetworkBot()
        self.genome_size = len(example_bot.get_weights())

    def evaluate_fitness(self, bot):
        """Оценивает качество бота"""
        total_score = 0
        games = 50

        opponents = [
            RandomBot(),
            RuleBasedBot(),
            MinimaxBot('easy')
        ]

        for _ in range(games):
            # Случайно выбираем противника
            opponent = random.choice(opponents)

            # Играем как X и как O
            score1 = self.play_game(bot, opponent, bot_player='O')
            score2 = self.play_game(opponent, bot, bot_player='X')

            total_score += score1 + score2

        return total_score / (games * 2)

    def play_game(self, player1, player2, bot_player='O'):
        """Играет партию и возвращает результат для бота"""
        board = [[' ' for _ in range(3)] for _ in range(3)]
        current_player = 'X'
        moves = 0

        while moves < 9:
            if current_player == 'X':
                move = player1.make_move(board, 'X') if hasattr(player1, 'make_move') else player1.make_move(board)
            else:
                move = player2.make_move(board, 'O') if hasattr(player2, 'make_move') else player2.make_move(board)

            if move is None:
                break

            board[move[0]][move[1]] = current_player
            moves += 1

            winner = self.check_winner(board)
            if winner:
                if winner == bot_player:
                    return 1.0  # Победа
                else:
                    return -0.5  # Поражение

            current_player = 'O' if current_player == 'X' else 'X'

        return 0.1  # Ничья

    def check_winner(self, board):
        """Проверяет победителя"""
        lines = []
        for i in range(3):
            lines.append([board[i][j] for j in range(3)])
            lines.append([board[j][i] for j in range(3)])
        lines.append([board[i][i] for i in range(3)])
        lines.append([board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]
        return None

    def crossover(self, parent1, parent2):
        """Скрещивание двух родителей"""
        weights1 = parent1.get_weights()
        weights2 = parent2.get_weights()

        # Случайная точка скрещивания
        crossover_point = random.randint(0, len(weights1))

        child_weights = np.concatenate([
            weights1[:crossover_point],
            weights2[crossover_point:]
        ])

        return NeuralNetworkBot(child_weights)

    def mutate(self, bot, mutation_rate=0.1, mutation_strength=0.1):
        """Мутирует бота"""
        weights = bot.get_weights()

        for i in range(len(weights)):
            if random.random() < mutation_rate:
                weights[i] += random.gauss(0, mutation_strength)

        return NeuralNetworkBot(weights)

    def evolve(self):
        """Один шаг эволюции"""
        print(f"Оценка поколения {self.generation}...")

        # Оценяем всю популяцию
        fitness_scores = []
        for i, bot in enumerate(self.population):
            fitness = self.evaluate_fitness(bot)
            fitness_scores.append((bot, fitness))

            if i % 10 == 0:
                print(f"Оценено {i+1}/{self.population_size} ботов")

        fitness_scores.sort(key=lambda x: x[1], reverse=True)

        # Статистика
        best_fitness = fitness_scores[0][1]
        avg_fitness = sum(score for _, score in fitness_scores) / len(fitness_scores)

        print(f"Поколение {self.generation}:")
        print(f"  Лучший результат: {best_fitness:.3f}")
        print(f"  Средний результат: {avg_fitness:.3f}")

        # Отбор лучших 30%
        elite_count = self.population_size // 3
        elite = [bot for bot, _ in fitness_scores[:elite_count]]

        # Создаём новое поколение
        new_population = elite.copy()  # Элита переходит без изменений

        while len(new_population) < self.population_size:
            # Турнирный отбор родителей
            parent1 = self.tournament_selection(fitness_scores)
            parent2 = self.tournament_selection(fitness_scores)

            # Скрещивание
            child = self.crossover(parent1, parent2)

            # Мутация
            child = self.mutate(child)

            new_population.append(child)

        self.population = new_population
        self.generation += 1

        return fitness_scores[0][0]  # Возвращаем лучшего бота

    def tournament_selection(self, fitness_scores, tournament_size=5):
        """Турнирный отбор"""
        tournament = random.sample(fitness_scores, tournament_size)
        return max(tournament, key=lambda x: x[1])[0]

## 🎮 Практическое применение: полная игра

**Соберём всё вместе в функциональную игру:**

```python
class TicTacToeGame:
    """Полная реализация игры в крестики-нолики с разными ИИ"""

    def __init__(self):
        self.board = [[' ' for _ in range(3)] for _ in range(3)]
        self.available_bots = {
            'random': RandomBot(),
            'rules': RuleBasedBot(),
            'minimax_easy': MinimaxBot('easy'),
            'minimax_hard': MinimaxBot('hard'),
            'qlearning': QLearningBot(),
            'neural': NeuralNetworkBot()
        }

    def reset_board(self):
        """Сбрасывает доску"""
        self.board = [[' ' for _ in range(3)] for _ in range(3)]

    def print_board(self):
        """Печатает текущее состояние доски"""
        print("\n   0   1   2")
        for i in range(3):
            print(f"{i}  {self.board[i][0]} | {self.board[i][1]} | {self.board[i][2]}")
            if i < 2:
                print("  ---|---|---")
        print()

    def human_move(self):
        """Получает ход от человека"""
        while True:
            try:
                move = input("Ваш ход (строка столбец, например '1 1'): ").strip()
                if move.lower() in ['quit', 'q']:
                    return None

                row, col = map(int, move.split())

                if 0 <= row <= 2 and 0 <= col <= 2:
                    if self.board[row][col] == ' ':
                        return (row, col)
                    else:
                        print("Эта клетка уже занята!")
                else:
                    print("Координаты должны быть от 0 до 2!")

            except (ValueError, IndexError):
                print("Неверный формат! Введите два числа через пробел.")

    def make_move(self, row, col, player):
        """Делает ход"""
        if self.board[row][col] == ' ':
            self.board[row][col] = player
            return True
        return False

    def check_winner(self):
        """Проверяет победителя"""
        lines = []

        # Строки и столбцы
        for i in range(3):
            lines.append([self.board[i][j] for j in range(3)])
            lines.append([self.board[j][i] for j in range(3)])

        # Диагонали
        lines.append([self.board[i][i] for i in range(3)])
        lines.append([self.board[i][2-i] for i in range(3)])

        for line in lines:
            if line[0] == line[1] == line[2] and line[0] != ' ':
                return line[0]

        return None

    def is_board_full(self):
        """Проверяет, заполнена ли доска"""
        return all(self.board[i][j] != ' ' for i in range(3) for j in range(3))

    def play_vs_ai(self, ai_type='minimax_hard'):
        """Игра человека против ИИ"""
        if ai_type not in self.available_bots:
            print(f"Неизвестный тип ИИ: {ai_type}")
            return

        ai_bot = self.available_bots[ai_type]
        self.reset_board()

        print(f"Игра против {ai_type} ИИ!")
        print("Вы играете за X, ИИ за O")
        print("Для выхода введите 'quit'")

        current_player = 'X'  # Человек начинает

        while True:
            self.print_board()

            if current_player == 'X':  # Ход человека
                print("Ваш ход!")
                move = self.human_move()

                if move is None:  # Выход
                    print("Спасибо за игру!")
                    return

                self.make_move(move[0], move[1], 'X')

            else:  # Ход ИИ
                print("ИИ думает...")

                # Специальная обработка для Q-Learning бота
                if isinstance(ai_bot, QLearningBot):
                    move = ai_bot.make_move(self.board, 'O')
                else:
                    move = ai_bot.make_move(self.board)

                if move:
                    self.make_move(move[0], move[1], 'O')
                    print(f"ИИ сходил на позицию {move}")
                else:
                    print("ИИ не может сделать ход!")
                    break

            # Проверяем результат
            winner = self.check_winner()
            if winner:
                self.print_board()
                if winner == 'X':
                    print("🎉 Поздравляем! Вы победили!")
                else:
                    print("🤖 ИИ победил! Попробуйте ещё раз.")
                break
            elif self.is_board_full():
                self.print_board()
                print("🤝 Ничья! Хорошая игра!")
                break

            # Смена игрока
            current_player = 'O' if current_player == 'X' else 'X'

    def ai_vs_ai_tournament(self, rounds=100):
        """Турнир между разными ИИ"""
        print("🏆 Турнир ИИ ботов!")
        print("=" * 50)

        results = {}
        bots = list(self.available_bots.items())

        # Каждый против каждого
        for i, (name1, bot1) in enumerate(bots):
            for j, (name2, bot2) in enumerate(bots):
                if i >= j:  # Избегаем дубликатов
                    continue

                print(f"Матч: {name1} vs {name2}")

                wins1 = wins2 = draws = 0

                for round_num in range(rounds):
                    # Игра 1: bot1 как X, bot2 как O
                    self.reset_board()
                    result = self.simulate_game(bot1, bot2)

                    if result == 'X':
                        wins1 += 1
                    elif result == 'O':
                        wins2 += 1
                    else:
                        draws += 1

                    # Игра 2: bot2 как X, bot1 как O
                    self.reset_board()
                    result = self.simulate_game(bot2, bot1)

                    if result == 'X':
                        wins2 += 1
                    elif result == 'O':
                        wins1 += 1
                    else:
                        draws += 1

                total_games = rounds * 2
                print(f"  {name1}: {wins1} побед ({wins1/total_games*100:.1f}%)")
                print(f"  {name2}: {wins2} побед ({wins2/total_games*100:.1f}%)")
                print(f"  Ничьи: {draws} ({draws/total_games*100:.1f}%)")
                print()

                # Сохраняем результаты
                if name1 not in results:
                    results[name1] = {'wins': 0, 'games': 0}
                if name2 not in results:
                    results[name2] = {'wins': 0, 'games': 0}

                results[name1]['wins'] += wins1
                results[name1]['games'] += total_games
                results[name2]['wins'] += wins2
                results[name2]['games'] += total_games

        # Итоговые результаты
        print("🏅 ИТОГОВЫЕ РЕЗУЛЬТАТЫ:")
        print("=" * 50)

        rankings = []
        for name, stats in results.items():
            win_rate = stats['wins'] / stats['games'] * 100
            rankings.append((name, win_rate, stats['wins'], stats['games']))

        rankings.sort(key=lambda x: x[1], reverse=True)

        for rank, (name, win_rate, wins, games) in enumerate(rankings, 1):
            print(f"{rank}. {name:15} - {win_rate:5.1f}% ({wins}/{games})")

    def simulate_game(self, bot1, bot2):
        """Симулирует игру между двумя ботами"""
        current_player = 'X'
        moves = 0

        while moves < 9:
            if current_player == 'X':
                bot = bot1
            else:
                bot = bot2

            # Получаем ход от бота
            if isinstance(bot, QLearningBot):
                move = bot.make_move(self.board, current_player)
            else:
                move = bot.make_move(self.board)

            if move is None:
                break

            self.make_move(move[0], move[1], current_player)
            moves += 1

            winner = self.check_winner()
            if winner:
                return winner

            current_player = 'O' if current_player == 'X' else 'X'

        return 'Draw'

# Пример использования
if __name__ == "__main__":
    game = TicTacToeGame()

    print("Добро пожаловать в крестики-нолики с ИИ!")
    print("\nДоступные противники:")
    print("1. random - Случайный бот")
    print("2. rules - Бот с правилами")
    print("3. minimax_easy - Лёгкий Minimax")
    print("4. minimax_hard - Сложный Minimax")
    print("5. qlearning - Обучающийся бот")
    print("6. neural - Нейросетевой бот")
    print("7. tournament - Турнир ИИ")

    choice = input("\nВыберите режим (введите номер): ").strip()

    if choice == '7':
        game.ai_vs_ai_tournament()
    else:
        ai_types = ['', 'random', 'rules', 'minimax_easy', 'minimax_hard', 'qlearning', 'neural']

        if choice.isdigit() and 1 <= int(choice) <= 6:
            ai_type = ai_types[int(choice)]
            game.play_vs_ai(ai_type)
        else:
            print("Неверный выбор!")
```

## 🚀 Заключение и дальнейшее развитие

### Что мы изучили:

- **Базовые техники:** FSM, деревья поведений
- **Классические алгоритмы:** Minimax, Alpha-Beta
- **Современные подходы:** MCTS, нейросети
- **Обучение:** Q-Learning, генетические алгоритмы
- **Оптимизации:** эвристики, кеширование

### Куда развиваться дальше:

- **Более сложные игры:** шахматы, шашки, Go
- **Real-time стратегии:** StarCraft, Age of Empires
- **Процедурная генерация:** создание уровней, контента
- **Эмоциональный ИИ:** персонажи с настроением
- **Кооперативный ИИ:** командная работа ботов

### Полезные ресурсы:

- **"AI for Games"** - Ian Millington
- **"Game AI Pro"** - серия книг
- **OpenAI Gym** - среда для обучения ИИ
- **Unity ML-Agents** - машинное обучение в Unity

---

**Игровой ИИ — это увлекательная область, где можно экспериментировать с разными подходами и видеть результаты в действии!** 🎮🤖

---

Автор: legenda  
Telegram: [@LEGENDA_SD](https://t.me/LEGENDA_SD)
