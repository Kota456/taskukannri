# taskukannri
import React, { useState, useEffect, createContext, useContext } from 'react';
import { Users, Plus, Filter, Share2, LogOut, Clock, AlertCircle, CheckCircle, Circle } from 'lucide-react';

// Team Context
const TeamContext = createContext();

const useTeam = () => {
  const context = useContext(TeamContext);
  if (!context) {
    throw new Error('useTeam must be used within a TeamProvider');
  }
  return context;
};

const TeamProvider = ({ children }) => {
  const [teams, setTeams] = useState({});
  const [currentTeam, setCurrentTeam] = useState(null);

  // データの保存（本番環境ではFirebase等を使用）
  const saveTeamData = async (teamCode, data) => {
    try {
      const allTeams = JSON.parse(localStorage.getItem('allTeams') || '{}');
      allTeams[teamCode.toUpperCase()] = {
        ...data,
        lastUpdated: new Date().toISOString(),
        version: (data.version || 0) + 1
      };
      localStorage.setItem('allTeams', JSON.stringify(allTeams));
      setTeams(allTeams);
      return allTeams[teamCode.toUpperCase()];
    } catch (error) {
      console.error('Failed to save team data:', error);
      throw error;
    }
  };

  const loadTeamData = async (teamCode) => {
    try {
      const allTeams = JSON.parse(localStorage.getItem('allTeams') || '{}');
      return allTeams[teamCode.toUpperCase()] || null;
    } catch (error) {
      console.error('Failed to load team data:', error);
      return null;
    }
  };

  const generateTeamCode = () => {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let code = '';
    for (let i = 0; i < 6; i++) {
      code += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return code.toUpperCase();
  };

  const value = {
    teams,
    currentTeam,
    setCurrentTeam,
    saveTeamData,
    loadTeamData,
    generateTeamCode
  };

  return (
    <TeamContext.Provider value={value}>
      {children}
    </TeamContext.Provider>
  );
};

// ホーム画面コンポーネント
const HomePage = ({ onTeamSelect }) => {
  const [teamName, setTeamName] = useState('');
  const [joinCode, setJoinCode] = useState('');
  const [isCreating, setIsCreating] = useState(false);
  const [isJoining, setIsJoining] = useState(false);
  
  const { generateTeamCode, saveTeamData, loadTeamData } = useTeam();

  const createTeam = async () => {
    if (!teamName.trim()) {
      alert('チーム名を入力してください');
      return;
    }

    try {
      const code = generateTeamCode();
      const teamData = {
        code,
        name: teamName.trim(),
        tasks: [],
        createdAt: new Date().toISOString(),
        version: 1
      };

      await saveTeamData(code, teamData);
      onTeamSelect(teamData);
      alert(`チーム "${teamData.name}" を作成しました！`);
      setIsCreating(false);
      setTeamName('');
    } catch (error) {
      alert('チーム作成に失敗しました');
    }
  };

  const joinTeam = async () => {
    if (!joinCode.trim()) {
      alert('チームコードを入力してください');
      return;
    }

    try {
      const team = await loadTeamData(joinCode.trim());
      if (!team) {
        alert('指定されたチームコードが見つかりません');
        return;
      }

      onTeamSelect(team);
      alert(`チーム "${team.name}" に参加しました！`);
      setIsJoining(false);
      setJoinCode('');
    } catch (error) {
      alert('チーム参加に失敗しました');
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center p-4">
      <div className="bg-white p-8 rounded-2xl shadow-xl max-w-md w-full">
        <div className="text-center mb-8">
          <Users className="mx-auto h-12 w-12 text-blue-600 mb-4" />
          <h1 className="text-3xl font-bold text-gray-800 mb-2">
            チームタスクマネージャー
          </h1>
          <p className="text-gray-600">チームでタスクを共有・管理しましょう</p>
        </div>
        
        <div className="space-y-4">
          <button
            onClick={() => setIsCreating(true)}
            className="w-full bg-blue-600 text-white py-3 px-4 rounded-lg hover:bg-blue-700 transition-colors font-medium flex items-center justify-center gap-2"
          >
            <Plus className="h-5 w-5" />
            新しいチームを作成
          </button>
          
          <button
            onClick={() => setIsJoining(true)}
            className="w-full bg-green-600 text-white py-3 px-4 rounded-lg hover:bg-green-700 transition-colors font-medium flex items-center justify-center gap-2"
          >
            <Users className="h-5 w-5" />
            既存のチームに参加
          </button>
        </div>

        {/* チーム作成モーダル */}
        {isCreating && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white p-6 rounded-lg shadow-xl max-w-md w-full mx-4">
              <h3 className="text-lg font-semibold mb-4 text-gray-800">新しいチームを作成</h3>
              <input
                type="text"
                value={teamName}
                onChange={(e) => setTeamName(e.target.value)}
                placeholder="チーム名を入力してください"
                className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 mb-4"
                onKeyPress={(e) => e.key === 'Enter' && createTeam()}
              />
              <div className="flex gap-3 justify-end">
                <button
                  onClick={() => {
                    setIsCreating(false);
                    setTeamName('');
                  }}
                  className="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition-colors"
                >
                  キャンセル
                </button>
                <button
                  onClick={createTeam}
                  className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 transition-colors"
                >
                  作成
                </button>
              </div>
            </div>
          </div>
        )}

        {/* チーム参加モーダル */}
        {isJoining && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white p-6 rounded-lg shadow-xl max-w-md w-full mx-4">
              <h3 className="text-lg font-semibold mb-4 text-gray-800">チームに参加</h3>
              <input
                type="text"
                value={joinCode}
                onChange={(e) => setJoinCode(e.target.value.toUpperCase())}
                placeholder="チームコードを入力してください"
                className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 mb-4 text-center font-mono text-lg"
                onKeyPress={(e) => e.key === 'Enter' && joinTeam()}
              />
              <div className="flex gap-3 justify-end">
                <button
                  onClick={() => {
                    setIsJoining(false);
                    setJoinCode('');
                  }}
                  className="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition-colors"
                >
                  キャンセル
                </button>
                <button
                  onClick={joinTeam}
                  className="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 transition-colors"
                >
                  参加
                </button>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

// チームダッシュボードコンポーネント
const TeamDashboard = ({ team, onLeaveTeam }) => {
  const { loadTeamData, saveTeamData, setCurrentTeam } = useTeam();
  
  const [tasks, setTasks] = useState(team.tasks || []);
  const [lastSync, setLastSync] = useState(new Date().toLocaleTimeString());

  // タスク関連の状態
  const [title, setTitle] = useState('');
  const [assignee, setAssignee] = useState('');
  const [deadline, setDeadline] = useState('');
  const [status, setStatus] = useState('pending');
  const [priority, setPriority] = useState('medium');

  // フィルター・ソート
  const [filter, setFilter] = useState('all');
  const [assigneeFilter, setAssigneeFilter] = useState('all');
  const [sortBy, setSortBy] = useState('deadline');

  // UI状態
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(null);
  const [showShareModal, setShowShareModal] = useState(false);
  const [showTaskForm, setShowTaskForm] = useState(false);

  const assigneeOptions = ['黄葉', '山田', '三村', '西村', '中村', '水田'];

  // リアルタイム同期
  useEffect(() => {
    const interval = setInterval(async () => {
      try {
        const latest = await loadTeamData(team.code);
        if (latest && latest.version > team.version) {
          setTasks(latest.tasks || []);
          setCurrentTeam(latest);
          setLastSync(new Date().toLocaleTimeString());
        }
      } catch (err) {
        console.error('Sync error:', err);
      }
    }, 5000);

    return () => clearInterval(interval);
  }, [team, loadTeamData, setCurrentTeam]);

  // タスク追加
  const addTask = async () => {
    if (!title.trim()) {
      alert('タイトルを入力してください');
      return;
    }

    try {
      const newTask = {
        id: Date.now(),
        title: title.trim(),
        assignee: assignee || '未割当',
        deadline: deadline || '未設定',
        status,
        priority,
        createdAt: new Date().toISOString()
      };

      const updatedTasks = [...tasks, newTask];
      const updatedTeam = { ...team, tasks: updatedTasks };
      
      await saveTeamData(team.code, updatedTeam);
      setTasks(updatedTasks);
      setCurrentTeam(updatedTeam);

      // フォームリセット
      setTitle('');
      setAssignee('');
      setDeadline('');
      setStatus('pending');
      setPriority('medium');
      setShowTaskForm(false);
    } catch (err) {
      alert('タスクの追加に失敗しました');
    }
  };

  // タスク状態更新
  const updateTaskStatus = async (id, newStatus) => {
    try {
      const updatedTasks = tasks.map(task =>
        task.id === id ? { ...task, status: newStatus } : task
      );
      
      const updatedTeam = { ...team, tasks: updatedTasks };
      await saveTeamData(team.code, updatedTeam);
      setTasks(updatedTasks);
      setCurrentTeam(updatedTeam);
    } catch (err) {
      alert('タスクの更新に失敗しました');
    }
  };

  // タスク削除
  const deleteTask = async (id) => {
    try {
      const updatedTasks = tasks.filter(task => task.id !== id);
      const updatedTeam = { ...team, tasks: updatedTasks };
      
      await saveTeamData(team.code, updatedTeam);
      setTasks(updatedTasks);
      setCurrentTeam(updatedTeam);
      setShowDeleteConfirm(null);
    } catch (err) {
      alert('タスクの削除に失敗しました');
    }
  };

  // フィルタリング・ソート
  const getFilteredTasks = () => {
    let filtered = tasks;
    
    if (filter !== 'all') {
      filtered = filtered.filter(task => task.status === filter);
    }
    
    if (assigneeFilter !== 'all') {
      filtered = filtered.filter(task => task.assignee === assigneeFilter);
    }
    
    return filtered.sort((a, b) => {
      if (sortBy === 'deadline') {
        if (a.deadline === '未設定' && b.deadline === '未設定') return 0;
        if (a.deadline === '未設定') return 1;
        if (b.deadline === '未設定') return -1;
        return new Date(a.deadline) - new Date(b.deadline);
      }
      if (sortBy === 'priority') {
        const priorityOrder = { 'high': 3, 'medium': 2, 'low': 1 };
        return priorityOrder[b.priority] - priorityOrder[a.priority];
      }
      return 0;
    });
  };

  const getStatusIcon = (status) => {
    switch (status) {
      case 'completed':
        return <CheckCircle className="h-5 w-5 text-green-600" />;
      case 'in_progress':
        return <Clock className="h-5 w-5 text-blue-600" />;
      default:
        return <Circle className="h-5 w-5 text-gray-400" />;
    }
  };

  const getPriorityColor = (priority) => {
    switch (priority) {
      case 'high':
        return 'bg-red-100 text-red-800';
      case 'medium':
        return 'bg-yellow-100 text-yellow-800';
      case 'low':
        return 'bg-green-100 text-green-800';
      default:
        return 'bg-gray-100 text-gray-800';
    }
  };

  const filteredTasks = getFilteredTasks();

  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <div className="max-w-7xl mx-auto">
        {/* ヘッダー */}
        <div className="bg-white rounded-lg shadow-sm p-6 mb-6">
          <div className="flex justify-between items-center">
            <div>
              <h1 className="text-2xl font-bold text-gray-800 flex items-center gap-2">
                <Users className="h-8 w-8 text-blue-600" />
                {team.name}
              </h1>
              <p className="text-gray-600">チームコード: {team.code}</p>
              <p className="text-sm text-gray-500">最終同期: {lastSync}</p>
            </div>
            <div className="flex gap-2">
              <button
                onClick={() => setShowShareModal(true)}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors flex items-center gap-2"
              >
                <Share2 className="h-4 w-4" />
                共有
              </button>
              <button
                onClick={onLeaveTeam}
                className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition-colors flex items-center gap-2"
              >
                <LogOut className="h-4 w-4" />
                退出
              </button>
            </div>
          </div>
        </div>

        {/* タスク統計 */}
        <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6">
          <div className="bg-white p-4 rounded-lg shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-gray-600">総タスク数</p>
                <p className="text-2xl font-bold text-gray-800">{tasks.length}</p>
              </div>
              <div className="bg-blue-100 p-3 rounded-full">
                <Circle className="h-6 w-6 text-blue-600" />
              </div>
            </div>
          </div>
          <div className="bg-white p-4 rounded-lg shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-gray-600">進行中</p>
                <p className="text-2xl font-bold text-blue-600">
                  {tasks.filter(t => t.status === 'in_progress').length}
                </p>
              </div>
              <div className="bg-blue-100 p-3 rounded-full">
                <Clock className="h-6 w-6 text-blue-600" />
              </div>
            </div>
          </div>
          <div className="bg-white p-4 rounded-lg shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-gray-600">完了済み</p>
                <p className="text-2xl font-bold text-green-600">
                  {tasks.filter(t => t.status === 'completed').length}
                </p>
              </div>
              <div className="bg-green-100 p-3 rounded-full">
                <CheckCircle className="h-6 w-6 text-green-600" />
              </div>
            </div>
          </div>
          <div className="bg-white p-4 rounded-lg shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-gray-600">高優先度</p>
                <p className="text-2xl font-bold text-red-600">
                  {tasks.filter(t => t.priority === 'high').length}
                </p>
              </div>
              <div className="bg-red-100 p-3 rounded-full">
                <AlertCircle className="h-6 w-6 text-red-600" />
              </div>
            </div>
          </div>
        </div>

        {/* コントロール */}
        <div className="bg-white rounded-lg shadow-sm p-6 mb-6">
          <div className="flex flex-wrap gap-4 items-center justify-between">
            <div className="flex flex-wrap gap-4">
              <button
                onClick={() => setShowTaskForm(true)}
                className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors flex items-center gap-2"
              >
                <Plus className="h-4 w-4" />
                新しいタスク
              </button>
              
              <div className="flex items-center gap-2">
                <Filter className="h-4 w-4 text-gray-600" />
                <select
                  value={filter}
                  onChange={(e) => setFilter(e.target.value)}
                  className="border border-gray-300 rounded px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                  <option value="all">全ての状態</option>
                  <option value="pending">未着手</option>
                  <option value="in_progress">進行中</option>
                  <option value="completed">完了</option>
                </select>
              </div>
              
              <select
                value={assigneeFilter}
                onChange={(e) => setAssigneeFilter(e.target.value)}
                className="border border-gray-300 rounded px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
              >
                <option value="all">全ての担当者</option>
                {assigneeOptions.map(name => (
                  <option key={name} value={name}>{name}</option>
                ))}
                <option value="未割当">未割当</option>
              </select>
            </div>
            
            <div className="flex items-center gap-2">
              <span className="text-sm text-gray-600">並び順:</span>
              <select
                value={sortBy}
                onChange={(e) => setSortBy(e.target.value)}
                className="border border-gray-300 rounded px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
              >
                <option value="deadline">締切日順</option>
                <option value="priority">優先度順</option>
              </select>
            </div>
          </div>
        </div>

        {/* タスクリスト */}
        <div className="bg-white rounded-lg shadow-sm">
          {filteredTasks.length === 0 ? (
            <div className="p-8 text-center text-gray-500">
              <Circle className="h-12 w-12 mx-auto mb-4 text-gray-300" />
              <p>タスクがありません</p>
              <p className="text-sm">新しいタスクを追加してください</p>
            </div>
          ) : (
            <div className="divide-y divide-gray-200">
              {filteredTasks.map(task => (
                <div key={task.id} className="p-6 hover:bg-gray-50 transition-colors">
                  <div className="flex items-start justify-between">
                    <div className="flex items-start gap-3 flex-1">
                      <button
                        onClick={() => updateTaskStatus(task.id, 
                          task.status === 'completed' ? 'pending' : 
                          task.status === 'pending' ? 'in_progress' : 'completed'
                        )}
                        className="mt-1"
                      >
                        {getStatusIcon(task.status)}
                      </button>
                      
                      <div className="flex-1">
                        <h3 className={`font-medium ${task.status === 'completed' ? 'line-through text-gray-500' : 'text-gray-800'}`}>
                          {task.title}
                        </h3>
                        <div className="flex flex-wrap gap-4 mt-2 text-sm">
                          <span className="text-gray-600">担当: {task.assignee}</span>
                          <span className="text-gray-600">締切: {task.deadline}</span>
                          <span className={`px-2 py-1 rounded-full text-xs ${getPriorityColor(task.priority)}`}>
                            {task.priority === 'high' ? '高' : task.priority === 'medium' ? '中' : '低'}
                          </span>
                        </div>
                      </div>
                    </div>
                    
                    <div className="flex gap-2">
                      <select
                        value={task.status}
                        onChange={(e) => updateTaskStatus(task.id, e.target.value)}
                        className="text-sm border border-gray-300 rounded px-2 py-1 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                      >
                        <option value="pending">未着手</option>
                        <option value="in_progress">進行中</option>
                        <option value="completed">完了</option>
                      </select>
                      
                      <button
                        onClick={() => setShowDeleteConfirm(task.id)}
                        className="text-red-600 hover:text-red-700 text-sm px-2 py-1 rounded hover:bg-red-50 transition-colors"
                      >
                        削除
                      </button>
                    </div>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>

        {/* タスク追加フォーム */}
        {showTaskForm && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white p-6 rounded-lg shadow-xl max-w-md w-full mx-4">
              <h3 className="text-lg font-semibold mb-4 text-gray-800">新しいタスクを追加</h3>
              <div className="space-y-4">
                <input
                  type="text"
                  value={title}
                  onChange={(e) => setTitle(e.target.value)}
                  placeholder="タスクのタイトル"
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                />
                
                <select
                  value={assignee}
                  onChange={(e) => setAssignee(e.target.value)}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                  <option value="">担当者を選択</option>
                  {assigneeOptions.map(name => (
                    <option key={name} value={name}>{name}</option>
                  ))}
                </select>
                
                <input
                  type="date"
                  value={deadline}
                  onChange={(e) => setDeadline(e.target.value)}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                />
                
                <select
                  value={priority}
                  onChange={(e) => setPriority(e.target.value)}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                  <option value="low">低優先度</option>
                  <option value="medium">中優先度</option>
                  <option value="high">高優先度</option>
                </select>
              </div>
              
              <div className="flex gap-3 justify-end mt-6">
                <button
                  onClick={() => setShowTaskForm(false)}
                  className="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition-colors"
                >
                  キャンセル
                </button>
                <button
                  onClick={addTask}
                  className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 transition-colors"
                >
                  追加
                </button>
              </div>
            </div>
          </div>
        )}

        {/* 削除確認 */}
        {showDeleteConfirm && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white p-6 rounded-lg shadow-xl max-w-md w-full mx-4">
              <h3 className="text-lg font-semibold mb-4 text-gray-800">タスクを削除</h3>
              <p className="text-gray-600 mb-6">このタスクを削除しますか？この操作は取り消せません。</p>
              <div className="flex gap-3 justify-end">
                <button
                  onClick={() => setShowDeleteConfirm(null)}
                  className="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition-colors"
                >
                  キャンセル
                </button>
                <button
                  onClick={() => deleteTask(showDeleteConfirm)}
                  className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition-colors"
                >
                  削除
                </button>
              </div>
            </div>
          </div>
        )}

        {/* 共有モーダル */}
        {showShareModal && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div className="bg-white p-6 rounded-lg shadow-xl max-w-md w-full mx-4">
              <h3 className="text-lg font-semibold mb-4 text-gray-800">チームを共有</h3>
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">チームコード</label>
                  <div className="flex gap-2">
                    <input
                      type="text"
                      value={team.code}
                      readOnly
                      className="flex-1 p-3 border border-gray-300 rounded-lg bg-gray-50 text-center font-mono text-lg"
                    />
                    <button
                      onClick={() => {
                        navigator.clipboard.writeText(team.code);
                        alert('チームコードをクリップボードにコピーしました！');
                      }}
                      className="px-3 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 transition-colors"
                    >
                      コピー
                    </button>
                  </div>
                </div>
                
                <p className="text-sm text-gray-600">
                  このチームコードを他のメンバーに共有してチームに招待してください。
                </p>
              </div>
              <div className="flex justify-end mt-6">
                <button
                  onClick={() => setShowShareModal(false)}
                  className="px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition-colors"
                >
                  閉じる
                </button>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

// メインアプリケーション
const App = () => {
  const [currentTeam, setCurrentTeam] = useState(null);

  const handleTeamSelect = (team) => {
    setCurrentTeam(team);
  };

  const handleLeaveTeam = () => {
    setCurrentTeam(null);
  };

  return (
    <TeamProvider>
      <div className="App">
        {currentTeam ? (
          <TeamDashboard team={currentTeam} onLeaveTeam={handleLeaveTeam} />
        ) : (
          <HomePage onTeamSelect={handleTeamSelect} />
        )}
      </div>
    </TeamProvider>
  );
};

export default App
