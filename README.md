# streak-flow
Track your daily habits, visualize your streaks, and crush your goals. A sleek and modern habit tracker built using React and Tailwind CSS.
import React, { useState, useEffect } from 'react';
import { Check, Plus, Trash2, Award, Calendar, Flame, CheckCircle } from 'lucide-react';

export default function App() {
  // Load initial habits from localStorage or default to a sample list
  const [habits, setHabits] = useState(() => {
    const saved = localStorage.getItem('streakflow_habits');
    return saved ? JSON.parse(saved) : [
      { id: 1, name: 'Drink 3L Water', category: 'Health', history: ['2026-06-08', '2026-06-09'] },
      { id: 2, name: 'Read 10 Pages', category: 'Mind', history: ['2026-06-09'] },
      { id: 3, name: '30 mins Workout', category: 'Fitness', history: [] }
    ];
  });

  const [newHabitName, setNewHabitName] = useState('');
  const [newHabitCategory, setNewHabitCategory] = useState('Health');

  // Sync habits to localStorage whenever they change
  useEffect(() => {
    localStorage.setItem('streakflow_habits', JSON.stringify(habits));
  }, habits);

  // Helper: Get today's date string (YYYY-MM-DD)
  const getTodayString = () => {
    const today = new Date();
    return today.toISOString().split('T')[0];
  };

  // Helper: Get yesterday's date string
  const getYesterdayString = () => {
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);
    return yesterday.toISOString().split('T')[0];
  };

  // Calculate current streak for a habit
  const calculateStreak = (history) => {
    if (!history || history.length === 0) return 0;
    
    // Sort history descending (newest first)
    const sortedHistory = [...history].sort((a, b) => new Date(b) - new Date(a));
    const today = getTodayString();
    const yesterday = getYesterdayString();
    
    // If the most recent completion wasn't today or yesterday, streak is broken (0)
    if (sortedHistory[0] !== today && sortedHistory[0] !== yesterday) {
      return 0;
    }
    
    let streak = 0;
    let checkDate = new Date(sortedHistory[0]);
    
    // Traverse backward to count consecutive days
    for (let i = 0; i < sortedHistory.length; i++) {
      const currentEntryStr = sortedHistory[i];
      const checkDateStr = checkDate.toISOString().split('T')[0];
      
      if (currentEntryStr === checkDateStr) {
        streak++;
        // Move checkDate back by 1 day for the next loop iteration
        checkDate.setDate(checkDate.getDate() - 1);
      } else if (new Date(currentEntryStr) < checkDate) {
        // Gap detected in the consecutive streak
        break;
      }
    }
    return streak;
  };

  // Toggle habit completion for today
  const toggleHabitToday = (id) => {
    const today = getTodayString();
    setHabits(habits.map(habit => {
      if (habit.id === id) {
        const isCompletedToday = habit.history.includes(today);
        const updatedHistory = isCompletedToday
          ? habit.history.filter(date => date !== today) // Remove today (uncheck)
          : [...habit.history, today]; // Add today (check)
        return { ...habit, history: updatedHistory };
      }
      return habit;
    }));
  };

  // Add a new habit
  const handleAddHabit = (e) => {
    e.preventDefault();
    if (!newHabitName.trim()) return;

    const newHabit = {
      id: Date.now(),
      name: newHabitName.trim(),
      category: newHabitCategory,
      history: []
    };

    setHabits([...habits, newHabit]);
    setNewHabitName('');
  };

  // Delete a habit
  const deleteHabit = (id) => {
    setHabits(habits.filter(habit => habit.id !== id));
  };

  // Global Stat Calculations
  const todayStr = getTodayString();
  const completedTodayCount = habits.filter(h => h.history.includes(todayStr)).length;
  const totalHabitsCount = habits.length;
  const completionRate = totalHabitsCount > 0 ? Math.round((completedTodayCount / totalHabitsCount) * 100) : 0;
  const highestStreak = habits.length > 0 ? Math.max(...habits.map(h => calculateStreak(h.history))) : 0;

  return (
    <div className="min-h-screen bg-slate-950 text-slate-100 antialiased font-sans p-4 sm:p-8">
      <div className="max-w-4xl mx-auto space-y-8">
        
        {/* Header */}
        <header className="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 pb-6 border-b border-slate-800">
          <div>
            <h1 className="text-3xl font-extrabold tracking-tight bg-gradient-to-r from-emerald-400 to-cyan-400 bg-clip-text text-transparent">
              StreakFlow
            </h1>
            <p className="text-sm text-slate-400 mt-1">Track daily habits, visualize streaks, crush goals.</p>
          </div>
          <div className="flex items-center gap-2 bg-slate-900 px-4 py-2 rounded-xl border border-slate-800">
            <Calendar className="w-4 h-4 text-emerald-400" />
            <span className="text-sm font-medium text-slate-300">
              {new Date().toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric' })}
            </span>
          </div>
        </header>

        {/* Dashboard Stats */}
        <section className="grid grid-cols-1 sm:grid-cols-3 gap-4">
          <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl flex items-center gap-4">
            <div className="p-3 bg-emerald-500/10 rounded-xl text-emerald-400">
              <CheckCircle className="w-6 h-6" />
            </div>
            <div>
              <p className="text-xs text-slate-400 font-medium uppercase tracking-wider">Today's Progress</p>
              <p className="text-2xl font-bold mt-0.5">{completedTodayCount} / {totalHabitsCount}</p>
            </div>
          </div>

          <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl flex items-center gap-4">
            <div className="p-3 bg-cyan-500/10 rounded-xl text-cyan-400">
              <Award className="w-6 h-6" />
            </div>
            <div>
              <p className="text-xs text-slate-400 font-medium uppercase tracking-wider">Completion Rate</p>
              <p className="text-2xl font-bold mt-0.5">{completionRate}%</p>
            </div>
          </div>

          <div className="bg-slate-900 border border-slate-800 p-5 rounded-2xl flex items-center gap-4">
            <div className="p-3 bg-amber-500/10 rounded-xl text-amber-400">
              <Flame className="w-6 h-6 animate-pulse" />
            </div>
            <div>
              <p className="text-xs text-slate-400 font-medium uppercase tracking-wider">Best Active Streak</p>
              <p className="text-2xl font-bold mt-0.5">{highestStreak} {highestStreak === 1 ? 'day' : 'days'}</p>
            </div>
          </div>
        </section>

        {/* Form & Habits Split View */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8 items-start">
          
          {/* Create Habit Form */}
          <section className="bg-slate-900 border border-slate-800 p-6 rounded-2xl space-y-4">
            <h2 className="text-lg font-semibold text-slate-200">Create New Habit</h2>
            <form onSubmit={handleAddHabit} className="space-y-4">
              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5 uppercase">Habit Name</label>
                <input
                  type="text"
                  value={newHabitName}
                  onChange={(e) => setNewHabitName(e.target.value)}
                  placeholder="e.g., Meditate, Journal..."
                  className="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-2.5 text-sm text-slate-100 focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>

              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5 uppercase">Category</label>
                <select
                  value={newHabitCategory}
                  onChange={(e) => setNewHabitCategory(e.target.value)}
                  className="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-2.5 text-sm text-slate-100 focus:outline-none focus:border-emerald-500 transition-colors appearance-none"
                >
                  <option value="Health">Health</option>
                  <option value="Fitness">Fitness</option>
                  <option value="Mind">Mind</option>
                  <option value="Work">Work</option>
                </select>
              </div>

              <button
                type="submit"
                className="w-full bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-600 hover:to-teal-700 text-slate-950 font-semibold py-2.5 px-4 rounded-xl text-sm flex items-center justify-center gap-2 shadow-lg shadow-emerald-500/10 transition-all active:scale-[0.98]"
              />
                <Plus className="w-4 h-4 stroke-[3]" /> Add Habit
              </button>
            </form>
          </section>

          {/* Habits Tracker List */}
          <section className="lg:col-span-2 space-y-4">
            <div className="flex justify-between items-center">
              <h2 className="text-lg font-semibold text-slate-200">Your Habits</h2>
              <span className="text-xs text-slate-500">{habits.length} total</span>
            </div>

            {habits.length === 0 ? (
              <div className="bg-slate-900/50 border border-dashed border-slate-800 rounded-2xl p-12 text-center text-slate-500">
                <CheckCircle className="w-8 h-8 mx-auto mb-3 opacity-40" />
                <p className="text-sm">No habits configured yet. Create one to kickstart your day!</p>
              </div>
            ) : (
              <div className="space-y-3">
                {habits.map((habit) => {
                  const isDoneToday = habit.history.includes(todayStr);
                  const currentStreak = calculateStreak(habit.history);

                  return (
                    <div
                      key={habit.id}
                      className={`group bg-slate-900 border transition-all duration-200 rounded-2xl p-4 flex items-center justify-between gap-4 ${
                        isDoneToday ? 'border-emerald-500/30 bg-emerald-950/5' : 'border-slate-800 hover:border-slate-700'
                      }`}
                    >
                      {/* Left: Habit Info */}
                      <div className="flex items-center gap-4">
                        {/* Checkbox trigger button */}
                        <button
                          onClick={() => toggleHabitToday(habit.id)}
                          className={`w-6 h-6 rounded-lg flex items-center justify-center border transition-all ${
                            isDoneToday
                              ? 'bg-emerald-500 border-emerald-500 text-slate-950'
                              : 'border-slate-700 text-transparent hover:border-emerald-500/50 hover:bg-emerald-500/5'
                          }`}
                        >
                          <Check className="w-4 h-4 stroke-[3]" />
                        </button>

                        <div>
                          <h3 className={`font-semibold text-sm sm:text-base transition-colors ${isDoneToday ? 'text-slate-400 line-through' : 'text-slate-100'}`}>
                            {habit.name}
                          </h3>
                          <span className="inline-block bg-slate-950 text-slate-400 text-[10px] uppercase tracking-wider font-semibold px-2 py-0.5 rounded-md mt-1 border border-slate-800">
                            {habit.category}
                          </span>
                        </div>
                      </div>

                      {/* Right: Streak Count & Delete Action */}
                      <div className="flex items-center gap-4">
                        <div className="flex items-center gap-1.5 bg-slate-950 border border-slate-800 px-2.5 py-1 rounded-xl">
                          <Flame className={`w-4 h-4 ${currentStreak > 0 ? 'text-amber-500 fill-amber-500/20' : 'text-slate-600'}`} />
                          <span className={`text-xs font-bold ${currentStreak > 0 ? 'text-slate-200' : 'text-slate-500'}`}>
                            {currentStreak}d
                          </span>
                        </div>

                        <button
                          onClick={() => deleteHabit(habit.id)}
                          className="text-slate-600 hover:text-rose-400 p-1.5 rounded-lg opacity-0 group-hover:opacity-100 focus:opacity-100 transition-opacity"
                          title="Delete habit"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </div>
                  );
                })}
              </div>
            )}
          </section>

        </div>

      </div>
    </div>
  );
}
