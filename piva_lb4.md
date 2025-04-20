```Пива расстояние Левенштейна
use std::io;

/// Читает строку и преобразует в вектор чисел
fn read_numbers() -> Vec<usize> {
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();
    input
        .trim()
        .split_whitespace()
        .filter_map(|s| s.parse().ok())
        .collect()
}

/// Читает строку и преобразует в вектор символов
fn read_chars(prompt: &str) -> Vec<char> {
    println!("{}", prompt);
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();
    input.trim().chars().collect()
}

/// Вывод таблицы DP
fn print_dp_table(dp: &Vec<Vec<i32>>, source: &[char], target: &[char]) {
    print!("    "); // отступ для угла таблицы
    for &t in target {
        print!("{:^5}", t);
    }
    println!();

    for (i, row) in dp.iter().enumerate() {
        if i == 0 {
            print!("  ");
        } else {
            print!("{:^3}", source[i - 1]);
        }
        for &cell in row {
            if cell == i32::MAX {
                print!("{:^5}", "INF");
            } else {
                print!("{:^5}", cell);
            }
        }
        println!();
    }
    println!("---");
}

/// Вычисляет расстояние Левенштейна с учётом проклятых символов
fn calculate_edit_distance(
    source: &[char],
    target: &[char],
    cursed_indices: &[usize],
    (replace_cost, insert_cost, delete_cost): (i32, i32, i32),
) -> Option<i32> {
    let m = source.len();
    let n = target.len();
    let mut dp = vec![vec![i32::MAX; n + 1]; m + 1];

    dp[0][0] = 0;

    // Инициализация первого столбца (удаление)
    for i in 1..=m {
        let is_cursed = cursed_indices.contains(&(i - 1));
        let ch = source[i - 1];
        if is_cursed && ch != 'U' {
            break;
        }
        if dp[i - 1][0] != i32::MAX {
            dp[i][0] = dp[i - 1][0] + delete_cost;
        }
    }

    // Инициализация первой строки (вставка)
    for j in 1..=n {
        if dp[0][j - 1] != i32::MAX {
            dp[0][j] = dp[0][j - 1] + insert_cost;
        }
    }

    // Основной алгоритм
    for i in 1..=m {
        for j in 1..=n {
            let s_ch = source[i - 1];
            let t_ch = target[j - 1];
            let is_cursed = cursed_indices.contains(&(i - 1));

            if s_ch == t_ch {
                if dp[i - 1][j - 1] != i32::MAX {
                    dp[i][j] = dp[i - 1][j - 1];
                }
            } else {
                // Замена
                if !(is_cursed && s_ch == 'U') && !is_cursed {
                    if dp[i - 1][j - 1] != i32::MAX {
                        dp[i][j] = dp[i][j].min(dp[i - 1][j - 1] + replace_cost);
                    }
                }

                // Вставка
                if dp[i][j - 1] != i32::MAX {
                    dp[i][j] = dp[i][j].min(dp[i][j - 1] + insert_cost);
                }

                // Удаление
                if (!is_cursed || (is_cursed && s_ch == 'U')) && dp[i - 1][j] != i32::MAX {
                    dp[i][j] = dp[i][j].min(dp[i - 1][j] + delete_cost);
                }
            }
        }

        // 🔍 Вывод после каждой строки
        println!("DP таблица после обработки source[0..{}]:", i);
        print_dp_table(&dp, source, target);
    }

    let result = dp[m][n];
    if result == i32::MAX {
        None
    } else {
        Some(result)
    }
}

fn main() {
    println!("Введите стоимости операций [замена вставка удаление] через пробел (например: 1 1 1):");
    let mut costs_input = String::new();
    io::stdin().read_line(&mut costs_input).unwrap();
    let costs: Vec<i32> = if costs_input.trim().is_empty() {
        vec![1, 1, 1]
    } else {
        costs_input
            .trim()
            .split_whitespace()
            .filter_map(|s| s.parse::<i32>().ok())
            .collect()
    };

    if costs.len() != 3 {
        eprintln!("Ошибка: нужно ввести ровно три числа.");
        return;
    }

    let source = read_chars("Введите первую строку (source):");
    let target = read_chars("Введите вторую строку (target):");

    println!("Введите индексы проклятых символов первой строки (начиная с 0), через пробел:");
    let cursed_indices = read_numbers();

    match calculate_edit_distance(&source, &target, &cursed_indices, (costs[0], costs[1], costs[2])) {
        Some(cost) => println!("Минимальная стоимость преобразования: {}", cost),
        None => println!("Невозможно преобразовать строку с заданными ограничениями."),
    }
}

```
