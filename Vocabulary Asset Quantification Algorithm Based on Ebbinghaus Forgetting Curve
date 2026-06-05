#include <iostream>
#include <vector>
#include <cmath>
#include <string>
#include <map>
#include <ctime>
#include <iomanip>

using namespace std;

// ==========================================
// 1. 数据结构定义
// ==========================================

// 单词状态机枚举
enum class WordState {
    NEW,        // 新词区
    LEARNING,   // 动态抗争区
    STABLE,     // 稳定区
    MASTERED    // 永久记忆区
};

// 单词结构体
struct Word {
    string id;
    string spell;
    time_t createdAt;       // 首次录入的绝对时间戳
    time_t lastReviewedAt;  // 最后一次复习的绝对时间戳
    int reviewCount;        // 成功复习的总次数 n
};

// ==========================================
// 2. 词汇资产引擎核心逻辑
// ==========================================

class MemoryEngine {
private:
    // 计算两个时间戳之间相差的天数
    static double getDeltaDays(time_t from, time_t to) {
        return difftime(to, from) / (60.0 * 60.0 * 24.0);
    }

public:
    // 计算单词在指定日期 (targetDate) 的绝对提取概率
    static double calculateProbability(const Word& word, time_t targetDate) {
        // 如果查询的时间在单词创建之前，概率为 0
        if (targetDate < word.createdAt) return 0.0;
        
        double deltaDays = getDeltaDays(word.lastReviewedAt, targetDate);
        if (deltaDays < 0) return 0.0; 
        if (deltaDays == 0) return 1.0; // 刚刚背完的瞬间是 100%

        int n = word.reviewCount;
        
        // 核心引擎：初始记忆强度 S0 = 0.9, 放大系数 alpha = 1.9
        double memoryStrength = 0.9 * pow(1.9, n); 
        double p = exp(-deltaDays / memoryStrength);

        // 永远受制于 0.05 的遗忘底噪
        return max(0.05, p); 
    }

    // 状态机路由：判断单词属于哪个资产池
    static WordState getWordState(const Word& word, time_t targetDate, double pValue) {
        double deltaDaysSinceCreate = getDeltaDays(word.createdAt, targetDate);
        int n = word.reviewCount;

        // 1. Mastered: 成功复习 5 次以上
        if (n >= 5) return WordState::MASTERED;

        // 2. New: 录入不足 24 小时，且从未复习过
        if (n == 0 && deltaDaysSinceCreate <= 1.0) return WordState::NEW;

        // 3. Stable: 至少复习过 3 次，且当前记忆概率还在 85% 以上
        if (n >= 3 && pValue >= 0.85) return WordState::STABLE;

        // 4. Learning: 其他所有情况都在泥潭里挣扎
        return WordState::LEARNING;
    }

    // 算法 A：计算某天的“活跃词汇总量” (YZ平面连续值)
    static double calculateActiveVocabulary(const vector<Word>& words, time_t targetDate) {
        double activeVocab = 0.0;
        
        for (const auto& w : words) {
            if (targetDate < w.createdAt) continue;

            double p = calculateProbability(w, targetDate);
            WordState state = getWordState(w, targetDate, p);

            if (state == WordState::MASTERED) {
                activeVocab += 0.98; // 永久资产稳健底盘
            } else if (state == WordState::NEW) {
                activeVocab += 0.33; // 新词初始折损
            } else {
                activeVocab += p;    // 动态资产实时呼吸
            }
        }
        return activeVocab;
    }

    // 算法 B：计算某天的“资产切层结构” (XZ平面离散值)
    static map<WordState, int> calculateAssetStratification(const vector<Word>& words, time_t targetDate) {
        map<WordState, int> stratification = {
            {WordState::NEW, 0},
            {WordState::LEARNING, 0},
            {WordState::STABLE, 0},
            {WordState::MASTERED, 0}
        };
        
        for (const auto& w : words) {
            if (targetDate < w.createdAt) continue;

            double p = calculateProbability(w, targetDate);
            WordState state = getWordState(w, targetDate, p);
            stratification[state]++; // 统计绝对个数
        }
        return stratification;
    }
    
    // 辅助函数：将状态枚举转为字符串
    static string stateToString(WordState state) {
        switch(state) {
            case WordState::NEW: return "New (新词区)";
            case WordState::LEARNING: return "Learning (抗争区)";
            case WordState::STABLE: return "Stable (稳定区)";
            case WordState::MASTERED: return "Mastered (永久区)";
            default: return "Unknown";
        }
    }
};

// ==========================================
// 3. 测试与展示
// ==========================================

// 辅助函数：生成相对今天的时间戳
time_t getDaysAgo(int days) {
    return time(nullptr) - (days * 24 * 60 * 60);
}

int main() {
    time_t today = time(nullptr);

    // 模拟构建一个用户词库 (涵盖四个状态)
    vector<Word> myWordbook = {
        // 词 1: 刚背完的新词 (New)
        {"1", "apple", getDaysAgo(0), getDaysAgo(0), 0},
        
        // 词 2: 背过1次，正在遗忘 (Learning) -> 2天前学，1天前复习
        {"2", "ubiquitous", getDaysAgo(2), getDaysAgo(1), 1},
        
        // 词 3: 忘了很久的词 (Learning 底噪) -> 10天前学，没复习
        {"3", "deteriorate", getDaysAgo(10), getDaysAgo(10), 0},
        
        // 词 4: 刚复习完的稳定词 (Stable) -> 7天前学，复习了3次，半天前刚复习
        {"4", "meticulous", getDaysAgo(7), today - (12 * 60 * 60), 3}, 
        
        // 词 5: 远古掌握词 (Mastered) -> 30天前学，复习了6次
        {"5", "photosynthesis", getDaysAgo(30), getDaysAgo(10), 6}
    };

    cout << "========================================\n";
    cout << "    🌍 英语语言资产评估系统启动 \n";
    cout << "========================================\n\n";

    // 1. 打印单个单词的状态和记忆概率
    cout << "--- 单词详情分析 ---\n";
    for (const auto& w : myWordbook) {
        double p = MemoryEngine::calculateProbability(w, today);
        WordState state = MemoryEngine::getWordState(w, today, p);
        
        cout << left << setw(16) << w.spell 
             << " | State: " << setw(18) << MemoryEngine::stateToString(state)
             << " | Raw Prob: " << fixed << setprecision(3) << p 
             << " | Reviews: " << w.reviewCount << "\n";
    }

    // 2. 统计 XZ 平面资产结构
    auto stratification = MemoryEngine::calculateAssetStratification(myWordbook, today);
    
    // 3. 计算 YZ 平面总活跃词汇量
    double activeVocab = MemoryEngine::calculateActiveVocabulary(myWordbook, today);

    cout << "\n========================================\n";
    cout << "    📊 你的全局词汇资产看板 (今天)\n";
    cout << "========================================\n";
    cout << "Total Words     (总录入) : " << myWordbook.size() << " 个\n";
    cout << "Active Vocab  (实时战斗力): " << fixed << setprecision(2) << activeVocab << " 个\n\n";
    
    cout << "--- 资产结构拆解 (XZ 平面) ---\n";
    cout << "💎 Mastered (永久区) : " << stratification[WordState::MASTERED] << " 个\n";
    cout << "🛡️ Stable   (稳定区) : " << stratification[WordState::STABLE] << " 个\n";
    cout << "🔥 Learning (抗争区) : " << stratification[WordState::LEARNING] << " 个\n";
    cout << "🌱 New      (新词区) : " << stratification[WordState::NEW] << " 个\n";
    cout << "========================================\n";

    return 0;
}怎么样
