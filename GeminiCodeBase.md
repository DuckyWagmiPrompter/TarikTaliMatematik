import React, { useState, useEffect, useCallback, useRef } from 'react';
import { Play, Trophy, HelpCircle, RefreshCw, X, Check, Award, Flame, Brain, PauseCircle, Trees, Star, Zap, Lightbulb, Pause, Users, Copy, Share2, LogOut, Link, AlertTriangle } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, doc, setDoc, getDoc, onSnapshot, updateDoc, increment, collection, arrayUnion } from 'firebase/firestore';

// --- Firebase Configuration (Safe Init) ---
let auth = null;
let db = null;
let firebaseError = false;

try {
  if (typeof __firebase_config !== 'undefined') {
    const firebaseConfig = JSON.parse(__firebase_config);
    const app = initializeApp(firebaseConfig);
    auth = getAuth(app);
    db = getFirestore(app);
  } else {
    console.warn("Firebase config not found. App running in offline/UI-only mode.");
    firebaseError = true;
  }
} catch (e) {
  console.error("Firebase initialization failed:", e);
  firebaseError = true;
}

const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// --- Assets & Styles ---

const CustomStyles = () => (
  <style>{`
    @import url('https://fonts.googleapis.com/css2?family=Patrick+Hand&family=Sniglet:wght@400;800&family=Bubblegum+Sans&display=swap');
    
    :root { --story-edge: #EBD9B4; --forest-green: #2D5A27; --berry: #D44D5C; --leafy: #74A12E; }
    .font-display { font-family: 'Bubblegum Sans', cursive; }
    .font-body { font-family: 'Patrick Hand', cursive; }
    .font-accent { font-family: 'Sniglet', cursive; }
    .storybook-page {
        background-color: #FDF5E6;
        background-image: radial-gradient(circle at 2px 2px, rgba(0,0,0,0.05) 1px, transparent 0);
        background-size: 24px 24px;
        box-shadow: 0 10px 0 var(--story-edge), 0 20px 30px rgba(0,0,0,0.1);
        border: 8px solid white;
        transform: rotate(-1deg);
    }
    .rope-whimsical {
        background-image: repeating-linear-gradient(-45deg, #634832 0px, #634832 15px, #8B5E3C 15px, #8B5E3C 30px);
        border-top: 4px solid rgba(255,255,255,0.2);
        border-bottom: 4px solid rgba(0,0,0,0.2);
        border-radius: 4px;
    }
    .hand-drawn-border { border: 3px solid #4a3728; border-radius: 255px 15px 225px 15px/15px 225px 15px 255px; }
    .sketch-button { border: 3px solid #4a3728; border-radius: 15px 50px 30px 5px / 50px 20px 50px 25px; transition: transform 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
    .sketch-button:hover { transform: scale(1.05) rotate(1deg); }
    .sketch-button:active { transform: scale(0.95); }
    .sketch-button:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
    .animate-bounce-slight { animation: bounce-slight 2s infinite ease-in-out; }
    @keyframes bounce-slight { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-5px); } }
    @keyframes shake {
        0%, 100% { transform: rotate(-1deg) translateX(0); }
        10%, 30%, 50%, 70%, 90% { transform: rotate(-1deg) translateX(-5px); }
        20%, 40%, 60%, 80% { transform: rotate(-1deg) translateX(5px); }
    }
    .animate-shake { animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both; }
  `}</style>
);

const ForestBackground = () => (
  <div className="absolute inset-0 z-0">
    <img 
      className="w-full h-full object-cover scale-110 opacity-90" 
      src="https://lh3.googleusercontent.com/aida-public/AB6AXuAaIiQo4OUV01ExZ9Da2DjhdlA6eYXU_P7CbUNWMEfebHObvo5vNxiowEDiMPn_n_wjRTUO6yiMfZzmiXkFQciP1pegzDSESb0dJcJhkkOMsS0geS_0m-gXcj3J1KqwrgDSkD_Zz6TBCpAeqnWq-aZRGftGCNxbKaurziA_qLaRuOlSTj78ytbWs2DewIK8L7hzCKk4ach7o84hY8XYGs_3WVuT7A8B57UgVjtT_RxQsMNQ7u9gjh1wTlx9Sm78zeRiBx5vI5yuUfA" 
      alt="Latar Belakang Hutan"
      onError={(e) => {
        e.target.style.display = 'none';
        e.target.parentElement.style.backgroundColor = '#2D5A27'; 
      }}
    />
    <div className="absolute inset-0 bg-gradient-to-b from-[#e8f5e9]/80 via-[#2D5A27]/20 to-[#2D5A27]/60"></div>
  </div>
);

// --- Custom SVG Characters (Embedded) ---

const VectorCharacter = ({ type, className }) => {
  // Styles for the sticker look
  const stickerFilter = "drop-shadow(0px 4px 2px rgba(0,0,0,0.1))";
  
  const characters = {
    shiba: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <path d="M20 30 L30 10 L50 30" fill="#E8995E" stroke="#4a3728" strokeWidth="2" />
        <path d="M80 30 L70 10 L50 30" fill="#E8995E" stroke="#4a3728" strokeWidth="2" />
        <circle cx="50" cy="50" r="38" fill="#E8995E" />
        <path d="M30 50 Q50 40 70 50 L70 75 Q50 90 30 75 Z" fill="#FFF5E6" />
        <circle cx="40" cy="55" r="4" fill="#4a3728" />
        <circle cx="60" cy="55" r="4" fill="#4a3728" />
        <path d="M45 65 Q50 68 55 65" stroke="#4a3728" strokeWidth="2" fill="none" />
        <rect x="25" y="80" width="50" height="15" rx="5" fill="#4CAF50" stroke="#4a3728" strokeWidth="2" />
      </svg>
    ),
    panda: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <circle cx="30" cy="30" r="12" fill="#333" />
        <circle cx="70" cy="30" r="12" fill="#333" />
        <circle cx="50" cy="50" r="38" fill="white" stroke="#4a3728" strokeWidth="1" />
        <ellipse cx="38" cy="52" rx="8" ry="10" fill="#333" transform="rotate(-20 38 52)" />
        <ellipse cx="62" cy="52" rx="8" ry="10" fill="#333" transform="rotate(20 62 52)" />
        <circle cx="39" cy="50" r="3" fill="white" />
        <circle cx="61" cy="50" r="3" fill="white" />
        <circle cx="50" cy="62" r="3" fill="#333" />
        <path d="M45 68 Q50 72 55 68" stroke="#333" strokeWidth="2" fill="none" />
        <rect x="20" y="80" width="60" height="15" rx="5" fill="#FF5722" stroke="#4a3728" strokeWidth="2" />
      </svg>
    ),
    bear: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <circle cx="25" cy="30" r="10" fill="#8D6E63" stroke="#4a3728" strokeWidth="2" />
        <circle cx="75" cy="30" r="10" fill="#8D6E63" stroke="#4a3728" strokeWidth="2" />
        <circle cx="50" cy="50" r="38" fill="#A1887F" />
        <ellipse cx="50" cy="60" rx="12" ry="10" fill="#D7CCC8" />
        <circle cx="40" cy="48" r="4" fill="#3E2723" />
        <circle cx="60" cy="48" r="4" fill="#3E2723" />
        <circle cx="50" cy="58" r="3" fill="#3E2723" />
        <rect x="30" y="82" width="40" height="12" rx="2" fill="#FFC107" stroke="#4a3728" strokeWidth="2" />
      </svg>
    ),
    rabbit: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <ellipse cx="35" cy="25" rx="8" ry="20" fill="#FFF" stroke="#4a3728" strokeWidth="2" transform="rotate(-10 35 25)" />
        <ellipse cx="65" cy="25" rx="8" ry="20" fill="#FFF" stroke="#4a3728" strokeWidth="2" transform="rotate(10 65 25)" />
        <circle cx="50" cy="55" r="35" fill="#FFF" />
        <circle cx="40" cy="50" r="4" fill="#333" />
        <circle cx="60" cy="50" r="4" fill="#333" />
        <path d="M45 60 L50 65 L55 60" stroke="#FF8A80" strokeWidth="2" fill="none" />
        <path d="M30 60 L15 55" stroke="#333" strokeWidth="1" />
        <path d="M70 60 L85 55" stroke="#333" strokeWidth="1" />
        <rect x="35" y="80" width="30" height="15" rx="4" fill="#9C27B0" stroke="#4a3728" strokeWidth="2" />
      </svg>
    ),
    cat: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <path d="M20 30 L30 10 L45 25" fill="#FFCC80" stroke="#4a3728" strokeWidth="2" />
        <path d="M80 30 L70 10 L55 25" fill="#333" stroke="#4a3728" strokeWidth="2" />
        <circle cx="50" cy="50" r="38" fill="#FFF" />
        {/* Patches */}
        <path d="M50 12 Q80 12 88 50 L50 50 Z" fill="#333" opacity="0.1" />
        <path d="M50 12 Q20 12 12 50 L50 50 Z" fill="#FFCC80" opacity="0.3" />
        
        <circle cx="40" cy="50" r="4" fill="#333" />
        <circle cx="60" cy="50" r="4" fill="#333" />
        <path d="M45 60 Q50 65 55 60" stroke="#333" strokeWidth="2" fill="none" />
        <rect x="20" y="80" width="60" height="15" rx="5" fill="#2196F3" stroke="#4a3728" strokeWidth="2" />
      </svg>
    ),
    penguin: (
      <svg viewBox="0 0 100 100" className={className} style={{ filter: stickerFilter }}>
        <circle cx="50" cy="50" r="46" fill="white" stroke="#4a3728" strokeWidth="3" />
        <circle cx="50" cy="50" r="38" fill="#263238" />
        <ellipse cx="50" cy="55" rx="30" ry="28" fill="white" />
        <circle cx="42" cy="45" r="4" fill="#333" />
        <circle cx="58" cy="45" r="4" fill="#333" />
        <path d="M45 52 L55 52 L50 60 Z" fill="#FFC107" stroke="#4a3728" strokeWidth="1" />
        <path d="M20 60 L10 50 L25 45" fill="#263238" stroke="#4a3728" strokeWidth="2" />
        <path d="M80 60 L90 50 L75 45" fill="#263238" stroke="#4a3728" strokeWidth="2" />
        <rect x="35" y="75" width="30" height="20" rx="2" fill="#F44336" stroke="#4a3728" strokeWidth="2" />
      </svg>
    )
  };

  return characters[type] || characters['shiba'];
};

const StickerAvatar = ({ type, alt, size = "w-28 h-28", isOpponent = false }) => {
  return (
    <div className={`relative ${size} transition-transform hover:scale-105 duration-200`}>
       <VectorCharacter type={type} className={`w-full h-full ${isOpponent ? 'grayscale-[0.5]' : ''}`} />
    </div>
  );
};

// --- Asset Collections (Using SVG Types) ---

const AVATAR_TYPES = ['shiba', 'panda', 'bear', 'rabbit'];
const MOTIVATOR_TYPES = ['cat', 'penguin', 'shiba'];

const MOTIVATIONAL_QUOTES = [
  "Anda Boleh!", "Teruskan Usaha!", "Mantap Sekali!", "Jangan Putus Asa!",
  "Fokus!", "Aum Macam Harimau!", "Sikit Lagi!", "Yakin Boleh!", "Tangkap Jawapan Itu!"
];

// --- Components ---

const Navbar = ({ score, gameState, isMultiplayer, roomName, onOpenLeaderboard, onOpenHowTo, copyShareLink }) => (
  <header className="fixed top-0 left-0 right-0 p-4 md:p-6 flex justify-between items-start z-50 pointer-events-none">
    <div className="flex flex-col md:flex-row gap-4 pointer-events-auto">
      <div className="bg-white/90 p-2 md:p-3 px-4 md:px-6 rounded-2xl hand-drawn-border flex items-center gap-4 shadow-md">
        <div className="bg-[#D44D5C]/20 p-2 rounded-full">
          <Star className="text-[#D44D5C]" size={32} />
        </div>
        <div>
          <p className="font-display text-lg text-[#D44D5C]/80 uppercase">Mata</p>
          <p className="font-display text-4xl text-[#2D5A27]">{score.toLocaleString()}</p>
        </div>
      </div>
    </div>
    <div className="flex gap-3 pointer-events-auto">
      {gameState === 'playing' && isMultiplayer && (
          <div className="bg-white/90 p-2 px-4 rounded-full hand-drawn-border flex items-center gap-2">
              <Users size={32} className="text-[#2D5A27]" />
              <span className="font-body font-bold hidden md:inline text-xl">{roomName}</span>
              <button onClick={copyShareLink} className="p-1 hover:text-green-600 bg-gray-100 rounded-full">
                  <Share2 size={24} />
              </button>
          </div>
      )}
      <button onClick={onOpenLeaderboard} className="bg-white/90 p-3 rounded-full hand-drawn-border hover:bg-[#74A12E]/10 transition-colors shadow-sm">
        <Trophy className="text-[#4a3728]" size={32} />
      </button>
      <button onClick={onOpenHowTo} className="bg-white/90 p-3 rounded-full hand-drawn-border hover:bg-[#74A12E]/10 transition-colors shadow-sm">
        <HelpCircle className="text-[#4a3728]" size={32} />
      </button>
    </div>
  </header>
);

const Modal = ({ title, children, onClose }) => (
  <div className="fixed inset-0 z-[60] flex items-center justify-center p-4 bg-black/50 backdrop-blur-sm">
    <div className="bg-[#FDF5E6] rounded-2xl shadow-2xl w-full max-w-md overflow-hidden relative hand-drawn-border p-6">
      <button onClick={onClose} className="absolute top-4 right-4 text-[#4a3728] hover:text-red-600">
        <X size={32} />
      </button>
      <h2 className="text-5xl font-display text-[#2D5A27] mb-4">{title}</h2>
      <div className="font-body text-2xl text-[#4a3728]">
          {children}
      </div>
    </div>
  </div>
);

const MotivatorCat = () => {
  const [visible, setVisible] = useState(false);
  const [message, setMessage] = useState('');
  const [currentType, setCurrentType] = useState(MOTIVATOR_TYPES[0]);

  useEffect(() => {
    const runCycle = () => {
      setMessage(MOTIVATIONAL_QUOTES[Math.floor(Math.random() * MOTIVATIONAL_QUOTES.length)]);
      setCurrentType(MOTIVATOR_TYPES[Math.floor(Math.random() * MOTIVATOR_TYPES.length)]);
      setVisible(true);
      setTimeout(() => setVisible(false), 4000);
    };

    let timeoutId = setTimeout(() => {
      runCycle();
      const intervalId = setInterval(runCycle, 15000); 
      return () => clearInterval(intervalId);
    }, 5000);
    return () => clearTimeout(timeoutId);
  }, []);

  return (
    <div className={`fixed left-0 bottom-24 z-50 transition-transform duration-500 ease-in-out transform ${visible ? 'translate-x-0' : '-translate-x-[120%]'}`}>
      <div className="relative pl-4">
        <div className={`absolute -top-24 left-20 bg-white border-2 border-[#4a3728] px-6 py-4 rounded-2xl shadow-[4px_4px_0_rgba(0,0,0,0.1)] transition-opacity duration-300 delay-300 ${visible ? 'opacity-100' : 'opacity-0'} animate-bounce-slight`}>
          <p className="font-display text-[#2D5A27] text-2xl whitespace-nowrap flex items-center gap-2">
             <Star size={24} className="text-[#FFD700] fill-current" />
             {message}
          </p>
          <div className="absolute -bottom-2 left-4 w-4 h-4 bg-white border-b-2 border-r-2 border-[#4a3728] transform rotate-45"></div>
        </div>
        <div className="w-28 h-28 md:w-36 md:h-36 relative">
             <VectorCharacter type={currentType} className="w-full h-full" />
        </div>
      </div>
    </div>
  );
};

// --- Game Constants ---
const GAME_DURATION_MS = 100;
const WIN_THRESHOLD = 90;
const LOSE_THRESHOLD = 10;
const START_POSITION = 50;

// --- Helper Functions ---
const generateQuestion = (difficulty) => {
  const operators = ['+', '-'];
  if (difficulty > 1) operators.push('×');
  const operator = operators[Math.floor(Math.random() * operators.length)];
  let num1, num2, answer;

  switch (operator) {
    case '+': num1 = Math.floor(Math.random() * (10 * difficulty)) + 5; num2 = Math.floor(Math.random() * (10 * difficulty)) + 5; answer = num1 + num2; break;
    case '-': num1 = Math.floor(Math.random() * (10 * difficulty)) + 10; num2 = Math.floor(Math.random() * num1) + 1; answer = num1 - num2; break;
    case '×': num1 = Math.floor(Math.random() * (3 * difficulty)) + 2; num2 = Math.floor(Math.random() * 9) + 2; answer = num1 * num2; break;
    default: answer = 0;
  }

  const options = new Set([answer]);
  while (options.size < 4) {
    const offset = Math.floor(Math.random() * 10) - 5;
    const wrong = answer + offset;
    if (wrong !== answer && wrong > 0) options.add(wrong);
    if (options.size < 4) options.add(answer + Math.floor(Math.random() * 20) + 1);
  }

  return { text: `${num1} ${operator} ${num2}`, answer, options: Array.from(options).sort(() => Math.random() - 0.5) };
};

export default function App() {
  const [user, setUser] = useState(null);
  const [isMultiplayer, setIsMultiplayer] = useState(false);
  const [lobbyState, setLobbyState] = useState('input'); 
  const [roomName, setRoomName] = useState('');
  const [playerName, setPlayerName] = useState('');
  const [roomData, setRoomData] = useState(null);
  const [playerRole, setPlayerRole] = useState(null);
  const [myAvatarType, setMyAvatarType] = useState(AVATAR_TYPES[0]);
  const [gameState, setGameState] = useState('menu');
  const [ropePosition, setRopePosition] = useState(START_POSITION);
  const [score, setScore] = useState(0);
  const [streak, setStreak] = useState(0);
  const [question, setQuestion] = useState(null);
  const [difficulty, setDifficulty] = useState(1);
  const [feedback, setFeedback] = useState(null);
  const [activeModal, setActiveModal] = useState(null);

  const timerRef = useRef(null);

  useEffect(() => {
    setMyAvatarType(AVATAR_TYPES[Math.floor(Math.random() * AVATAR_TYPES.length)]);
    const params = new URLSearchParams(window.location.search);
    const roomParam = params.get('room');
    if (roomParam) {
      setRoomName(roomParam);
      setIsMultiplayer(true);
      setLobbyState('input'); 
    }
  }, []);

  useEffect(() => {
    if (!auth) return;
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error("Auth failed:", err);
      }
    };
    initAuth();
    return onAuthStateChanged(auth, setUser);
  }, []);

  // Sync Room Data
  useEffect(() => {
    if (!user || !isMultiplayer || !roomName || !db) return;
    const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', roomName);
    const unsubscribe = onSnapshot(roomRef, (snapshot) => {
      if (snapshot.exists()) {
        const data = snapshot.data();
        setRoomData(data);
        if (data.status === 'playing') {
            if (gameState !== 'playing') startMultiplayerGame(data);
            const scoreDiff = (data.host.score || 0) - (data.guest?.score || 0);
            const rawPos = 50 + (scoreDiff * 2); 
            const clampedPos = Math.max(0, Math.min(100, rawPos));
            setRopePosition(clampedPos);
            
            // Check win condition for multiplayer
            if (clampedPos >= WIN_THRESHOLD) handleMultiplayerEnd('host');
            if (clampedPos <= LOSE_THRESHOLD) handleMultiplayerEnd('guest');
        } else if (data.status === 'finished') {
             if (data.winner === playerRole) setGameState('won');
             else setGameState('lost');
        }
      }
    }, (error) => console.error("Room sync error:", error));
    return () => unsubscribe();
  }, [user, isMultiplayer, roomName, gameState, playerRole]);

  // Check win condition for Single Player
  useEffect(() => {
    if (isMultiplayer) return;
    if (gameState !== 'playing') return;

    if (ropePosition >= WIN_THRESHOLD) {
      setGameState('won');
    } else if (ropePosition <= LOSE_THRESHOLD) {
      setGameState('lost');
    }
  }, [ropePosition, isMultiplayer, gameState]);

  const handleJoinRoom = async () => {
    if (!user || !roomName || !playerName || !db) return;
    const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', roomName);
    const roomSnap = await getDoc(roomRef);
    if (!roomSnap.exists()) {
      await setDoc(roomRef, {
        host: { id: user.uid, name: playerName, score: 0, avatar: myAvatarType }, // Share avatar type
        guest: null,
        status: 'waiting',
        currentTurn: 'host',
        createdAt: Date.now()
      });
      setPlayerRole('host');
      setLobbyState('waiting');
    } else {
      const data = roomSnap.data();
      if (!data.guest) {
        await updateDoc(roomRef, {
          guest: { id: user.uid, name: playerName, score: 0, avatar: myAvatarType }, // Share avatar type
          status: 'playing',
          currentTurn: 'host'
        });
        setPlayerRole('guest');
      } else {
        alert("Bilik penuh!");
        return;
      }
    }
    setScore(0);
    setDifficulty(1); 
  };

  const startMultiplayerGame = (data) => {
    setGameState('playing');
    setQuestion(generateQuestion(1));
  };

  const handleMultiplayerEnd = async (winnerRole) => {
      if (playerRole === 'host' && db) {
          const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', roomName);
          await updateDoc(roomRef, { status: 'finished', winner: winnerRole });
      }
  };

  const startSinglePlayer = () => {
    setIsMultiplayer(false);
    setGameState('playing');
    setRopePosition(START_POSITION);
    setScore(0);
    setStreak(0);
    setDifficulty(1);
    setQuestion(generateQuestion(1));
    setFeedback(null);
  };

  // Game Loop for Rope Physics (Single Player)
  useEffect(() => {
    if (gameState !== 'playing' || isMultiplayer) {
      if (timerRef.current) clearInterval(timerRef.current);
      return;
    }
    timerRef.current = setInterval(() => {
      setRopePosition((prev) => {
        const opponentPull = 0.15 + (difficulty * 0.05);
        const gravity = prev > 50 ? 0.05 : -0.05; // Rubber banding towards center
        let newPos = prev - opponentPull - gravity;
        return Math.max(0, Math.min(100, newPos));
      });
    }, GAME_DURATION_MS);
    return () => clearInterval(timerRef.current);
  }, [gameState, difficulty, isMultiplayer]);

  const handleAnswer = async (selectedOption) => {
    if (gameState !== 'playing') return;
    if (isMultiplayer && roomData && roomData.currentTurn !== playerRole) return; 

    if (selectedOption === question.answer) {
      setFeedback('correct');
      const newScore = score + 100;
      setScore(newScore); // Update score immediately
      setStreak(prev => prev + 1);

      if (isMultiplayer && db) {
          const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', roomName);
          const nextTurn = playerRole === 'host' ? 'guest' : 'host';
          if (playerRole === 'host') {
              await updateDoc(roomRef, { 'host.score': increment(10), currentTurn: nextTurn });
          } else {
              await updateDoc(roomRef, { 'guest.score': increment(10), currentTurn: nextTurn });
          }
      } else {
          // Single player pull
          const power = 8 + (streak * 0.5);
          setRopePosition((prev) => Math.min(prev + power, 100));
      }

      // Calculate next round difficulty immediately based on new score
      setTimeout(() => { 
        setFeedback(null); 
        const newDiff = Math.floor(newScore / 500) + 1;
        setDifficulty(newDiff);
        setQuestion(generateQuestion(newDiff));
      }, 400);

    } else {
      setFeedback('wrong');
      setStreak(0);
      if (!isMultiplayer) {
          const penalty = 10;
          setRopePosition((prev) => Math.max(prev - penalty, 0));
      } else if (db) {
          const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', roomName);
          const nextTurn = playerRole === 'host' ? 'guest' : 'host';
          await updateDoc(roomRef, { currentTurn: nextTurn });
      }
      setTimeout(() => setFeedback(null), 400);
    }
  };

  const visualRopePosition = (isMultiplayer && playerRole === 'guest') ? 100 - ropePosition : ropePosition;
  const isMyTurn = isMultiplayer ? (roomData?.currentTurn === playerRole) : true;
  
  const copyShareLink = () => {
      const url = new URL(window.location.href);
      url.searchParams.set('room', roomName);
      const text = url.toString();
      const textArea = document.createElement("textarea");
      textArea.value = text;
      textArea.style.position = "fixed";
      textArea.style.left = "-9999px";
      textArea.style.top = "0";
      document.body.appendChild(textArea);
      textArea.focus();
      textArea.select();
      try {
          const successful = document.execCommand('copy');
          if(successful) alert('Pautan jemputan disalin!');
      } catch (err) {
          console.error('Failed to copy', err);
          prompt("Sila salin pautan ini secara manual:", text);
      }
      document.body.removeChild(textArea);
  };

  if (firebaseError && isMultiplayer) {
    return (
      <div className="font-body bg-[#e8f5e9] min-h-screen flex items-center justify-center p-6 text-center">
        <CustomStyles />
        <div className="storybook-page p-10 max-w-md w-full">
            <AlertTriangle size={64} className="text-[#D44D5C] mx-auto mb-4" />
            <h2 className="text-5xl font-display text-[#4a3728] mb-4">Ralat Konfigurasi</h2>
            <p className="text-[#4a3728]/80 mb-8 text-2xl">Pangkalan data tidak dapat dihubungi. Sila main mod solo.</p>
            <button onClick={() => { setIsMultiplayer(false); setGameState('menu'); }} className="sketch-button bg-[#FFD700] text-[#2D5A27] font-display text-4xl py-4 px-10">Main Solo</button>
        </div>
      </div>
    );
  }

  // Get opponent avatar type safely
  const opponentAvatarType = isMultiplayer 
    ? (playerRole === 'host' ? roomData?.guest?.avatar : roomData?.host?.avatar) 
    : 'bear'; // Default AI opponent

  return (
    <div className="font-body bg-[#e8f5e9] text-[#2d4a22] min-h-screen overflow-hidden relative selection:bg-[#74A12E] selection:text-white">
      <CustomStyles />
      <ForestBackground />
      <Navbar 
        score={score} 
        gameState={gameState} 
        isMultiplayer={isMultiplayer} 
        roomName={roomName}
        onOpenLeaderboard={() => setActiveModal('leaderboard')}
        onOpenHowTo={() => setActiveModal('howTo')}
        copyShareLink={copyShareLink}
      />

      <main className="relative h-screen w-full flex flex-col items-center justify-center overflow-hidden">
        {gameState === 'menu' && !isMultiplayer && (
          <div className="absolute inset-0 z-40 flex items-center justify-center bg-black/20 backdrop-blur-sm">
            <div className="storybook-page p-12 max-w-2xl w-full text-center shadow-2xl m-4">
              <div className="mb-6 flex justify-center">
                 <div className="bg-[#74A12E]/20 p-6 rounded-full hand-drawn-border animate-bounce">
                    <Trees size={64} className="text-[#2D5A27]" />
                 </div>
              </div>
              <h1 className="font-display text-8xl text-[#2D5A27] mb-6 drop-shadow-sm">Tarik Tali <br/><span className="text-[#D44D5C]">Matematik</span></h1>
              <div className="flex flex-col gap-4 max-w-md mx-auto">
                  <button onClick={startSinglePlayer} className="sketch-button bg-[#FFD700] text-[#2D5A27] font-display text-5xl py-4 px-10 shadow-[0_6px_0_#d4a017] hover:brightness-110 active:shadow-none active:translate-y-[6px]">Main Solo</button>
                  <button onClick={() => { setIsMultiplayer(true); setLobbyState('input'); }} className="sketch-button bg-white text-[#2D5A27] font-display text-3xl py-4 px-10 shadow-[0_6px_0_#d1cfcf] hover:brightness-110 active:shadow-none active:translate-y-[6px] flex items-center justify-center gap-3"><Users size={32} />Main dengan Kawan</button>
              </div>
            </div>
          </div>
        )}

        {isMultiplayer && gameState === 'menu' && (
             <div className="absolute inset-0 z-40 flex items-center justify-center bg-black/40 backdrop-blur-sm">
                 <div className="storybook-page p-8 max-w-md w-full text-center shadow-2xl m-4 relative">
                     <button onClick={() => setIsMultiplayer(false)} className="absolute top-4 left-4 text-[#4a3728] hover:text-[#D44D5C]"><LogOut size={32} /></button>
                     <h2 className="font-display text-6xl text-[#2D5A27] mb-8">Jom Main Bersama!</h2>
                     {lobbyState === 'input' ? (
                         <div className="flex flex-col gap-6 text-left">
                             <div>
                                 <label className="font-accent font-bold text-2xl text-[#4a3728] ml-1">Nama Anda</label>
                                 <input type="text" className="w-full p-4 rounded-xl hand-drawn-border bg-white text-3xl font-body outline-none focus:ring-2 focus:ring-[#74A12E]" placeholder="Cth: Adli" value={playerName} onChange={(e) => setPlayerName(e.target.value)}/>
                             </div>
                             <div>
                                 <label className="font-accent font-bold text-2xl text-[#4a3728] ml-1">Nama Bilik</label>
                                 <input type="text" className="w-full p-4 rounded-xl hand-drawn-border bg-white text-3xl font-body outline-none focus:ring-2 focus:ring-[#74A12E]" placeholder="Cth: bilik-1" value={roomName} onChange={(e) => setRoomName(e.target.value)}/>
                                 <p className="text-xl text-[#4a3728]/70 mt-2 ml-1">Kawan anda boleh masuk dengan link yang sama.</p>
                             </div>
                             <button onClick={handleJoinRoom} disabled={!roomName || !playerName} className="mt-6 sketch-button bg-[#74A12E] text-white font-display text-4xl py-4 shadow-[0_6px_0_#2D5A27] active:shadow-none active:translate-y-[6px] disabled:opacity-50">Masuk Bilik</button>
                         </div>
                     ) : (
                         <div className="flex flex-col items-center gap-4">
                             <div className="w-24 h-24 rounded-full bg-[#FFD700] flex items-center justify-center hand-drawn-border animate-pulse"><Users size={40} className="text-[#2D5A27]" /></div>
                             <div>
                                <p className="font-body text-4xl font-bold text-[#4a3728]">Menunggu Kawan...</p>
                                <p className="text-2xl text-[#4a3728]/80">Bilik: <span className="font-bold text-[#D44D5C]">{roomName}</span></p>
                             </div>
                             <div className="bg-white/50 p-6 rounded-xl hand-drawn-border w-full">
                                <p className="font-bold text-3xl text-[#2D5A27]">{roomData?.host?.name || playerName} (Anda)</p>
                                <p className="text-[#4a3728]/50 italic text-xl mt-2">vs</p>
                                <p className="font-bold text-3xl text-[#4a3728]/50">Menunggu...</p>
                             </div>
                             <button onClick={copyShareLink} className="flex items-center gap-3 bg-[#74A12E] text-white px-6 py-3 rounded-xl font-bold text-2xl hover:bg-[#2D5A27] transition-colors shadow-md"><Link size={24} /> Salin Pautan Jemputan</button>
                         </div>
                     )}
                 </div>
             </div>
        )}

        <div className={`transition-opacity duration-500 w-full h-full flex flex-col items-center justify-center ${gameState === 'menu' ? 'opacity-0 pointer-events-none' : 'opacity-100'}`}>
            {gameState === 'playing' && <MotivatorCat />}
            {isMultiplayer && gameState === 'playing' && (
                <div className="absolute top-24 left-1/2 -translate-x-1/2 z-30">
                    <div className={`px-8 py-3 rounded-full hand-drawn-border shadow-lg font-display text-3xl transition-all duration-300 ${isMyTurn ? 'bg-[#74A12E] text-white scale-110' : 'bg-white text-[#4a3728] opacity-80'}`}>
                        {isMyTurn ? "Giliran Anda!" : `Giliran ${playerRole === 'host' ? roomData?.guest?.name || 'Lawan' : roomData?.host?.name || 'Lawan'}...`}
                    </div>
                </div>
            )}

            <div className="relative w-full h-64 flex items-center justify-center z-10 my-8">
                <div className="absolute w-full h-8 rope-whimsical shadow-[0_10px_20px_rgba(0,0,0,0.3)] flex items-center">
                    <div className="h-24 w-2 bg-[#D44D5C] absolute top-1/2 -translate-y-1/2 shadow-lg transition-all duration-300 ease-out z-20" style={{ left: `${visualRopePosition}%` }}>
                        <div className="absolute -top-10 -left-6 w-14 h-12 bg-[#D44D5C] rounded-sm flex items-center justify-center sketch-button rotate-[-5deg]">
                            <Star className="text-white fill-current" size={24} />
                        </div>
                    </div>
                </div>
                <div className="absolute left-4 lg:left-32 flex gap-4 lg:gap-6 items-end translate-y-[-20px] transition-transform duration-300" style={{ transform: `translateY(-20px) translateX(${(visualRopePosition - 50) * 0.5}px)` }}>
                    <div className="flex flex-col items-center">
                        <StickerAvatar type={myAvatarType} alt="Anda" />
                        <span className="mt-2 font-display text-xl bg-white px-4 py-1 rounded-full hand-drawn-border">{(isMultiplayer ? (playerRole === 'host' ? roomData?.host?.name : roomData?.guest?.name) : 'Anda') || 'Anda'}</span>
                    </div>
                </div>
                <div className="absolute right-4 lg:right-32 flex gap-4 lg:gap-6 items-end translate-y-[-20px] flex-row-reverse transition-transform duration-300" style={{ transform: `translateY(-20px) translateX(${(visualRopePosition - 50) * 0.5}px)` }}>
                    <div className="flex flex-col items-center">
                        <StickerAvatar type={opponentAvatarType} alt="Lawan" isOpponent={true} />
                        <span className="mt-2 font-display text-xl bg-white px-4 py-1 rounded-full hand-drawn-border">{(isMultiplayer ? (playerRole === 'host' ? roomData?.guest?.name : roomData?.host?.name) : 'Roh Bayang') || '...'}</span>
                    </div>
                </div>
            </div>

            <div className="relative z-30 w-full max-w-xl px-6 mb-12">
                <div className={`storybook-page p-6 md:p-10 rounded-xl relative transition-transform duration-200 ${feedback === 'wrong' ? 'animate-shake border-red-500' : ''}`}>
                    <div className="absolute left-0 top-0 bottom-0 w-8 bg-[var(--story-edge)] opacity-50 border-r border-black/10"></div>
                    <div className="ml-4">
                        <div className="flex flex-col md:flex-row justify-between items-center mb-8 gap-4">
                            <div className="relative w-28 h-28 flex items-center justify-center">
                                <svg className="w-full h-full transform -rotate-90" viewBox="0 0 80 80">
                                    <circle className="text-[#74A12E]/20" cx="40" cy="40" fill="transparent" r="34" stroke="currentColor" strokeWidth="8"></circle>
                                    <circle className="text-[#FFD700]" cx="40" cy="40" fill="transparent" r="34" stroke="currentColor" strokeDasharray="213" strokeDashoffset="50" strokeLinecap="round" strokeWidth="8"></circle>
                                </svg>
                                <div className="absolute flex flex-col items-center justify-center">
                                    <span className="text-xs font-accent font-bold opacity-60 text-[#2D5A27] uppercase tracking-wider">Soalan</span>
                                    <span className="font-display text-5xl text-[#2D5A27] leading-none pt-1">{Math.floor(score/100) + 1}</span>
                                </div>
                            </div>
                            <div className="text-center md:text-right">
                                <h2 className="font-display text-7xl tracking-wide text-[#3d2b1f]">{question ? question.text : '...'} = ?</h2>
                                <p className="font-accent text-2xl text-[#74A12E] mt-2 italic">{isMultiplayer ? (isMyTurn ? 'Giliran Anda!' : 'Tunggu lawan menjawab...') : 'Jawab untuk tarik lebih kuat!'}</p>
                            </div>
                        </div>
                        <div className="grid grid-cols-2 gap-4 md:gap-6">
                            {question && question.options.map((opt, i) => (
                                <button key={i} onClick={() => handleAnswer(opt)} disabled={isMultiplayer && !isMyTurn} className={`sketch-button font-display text-5xl md:text-6xl py-6 md:py-8 active:shadow-none active:translate-y-[6px] transition-colors ${feedback === 'correct' && opt === question.answer ? 'bg-[#74A12E] text-white shadow-[0_6px_0_#4a6e22]' : (isMultiplayer && !isMyTurn) ? 'bg-gray-100 text-gray-400 shadow-none' : 'bg-white text-[#2D5A27] shadow-[0_6px_0_#d1cfcf] hover:bg-[#FFD700] hover:shadow-[0_6px_0_#d4a017]'}`}>{opt}</button>
                            ))}
                        </div>
                    </div>
                </div>
            </div>

            <div className="hidden md:flex absolute bottom-8 left-1/2 -translate-x-1/2 w-full max-w-2xl px-8 z-20 flex-col items-center">
                <div className="flex justify-between w-full mb-2 px-2 text-white font-display text-3xl drop-shadow-md">
                    <span>{isMultiplayer ? 'Anda' : 'Wira Rimba'}</span>
                    <span>{isMultiplayer ? 'Lawan' : 'Roh Bayang'}</span>
                </div>
                <div className="w-full h-8 bg-[#3d2b1f]/40 rounded-full hand-drawn-border p-1.5 backdrop-blur-sm shadow-inner">
                    <div className="flex h-full rounded-full overflow-hidden">
                        <div className="h-full bg-[#74A12E] relative shadow-[inset_0_4px_10px_rgba(255,255,255,0.4)] transition-all duration-300" style={{ width: `${visualRopePosition}%` }}></div>
                        <div className="h-full bg-[#D44D5C] relative shadow-[inset_0_4px_10px_rgba(255,255,255,0.4)] transition-all duration-300" style={{ width: `${100 - visualRopePosition}%` }}></div>
                    </div>
                </div>
            </div>
            
            <div className="fixed bottom-6 right-6 bg-white/95 px-8 py-4 rounded-2xl flex items-center gap-4 shadow-xl hand-drawn-border">
                <Lightbulb className="text-[#74A12E]" size={32} /><span className="font-accent text-xl font-bold text-[#2D5A27] hidden md:inline">Cepat! Jawab pantas untuk Kuasa Rimba!</span>
            </div>
        </div>

        {(gameState === 'won' || gameState === 'lost') && (
           <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm">
             <div className="storybook-page p-10 max-w-md w-full text-center">
                <div className="mb-4 flex justify-center">
                  {gameState === 'won' ? ( <div className="bg-[#FFD700] p-6 rounded-full hand-drawn-border"><Award size={64} className="text-[#2D5A27]" /></div> ) : ( <div className="bg-[#D44D5C] p-6 rounded-full hand-drawn-border"><X size={64} className="text-white" /></div> )}
                </div>
                <h2 className="text-6xl font-display text-[#2D5A27] mb-4">{gameState === 'won' ? 'Kemenangan!' : 'Alamak!'}</h2>
                <p className="font-body text-3xl text-[#4a3728] mb-8">{gameState === 'won' ? `Hebat! Anda berjaya menarik kemenangan!` : `Lawan anda terlalu kuat kali ini.`}</p>
                <button onClick={() => { setGameState('menu'); setIsMultiplayer(false); setLobbyState('input'); setScore(0); }} className="w-full sketch-button bg-[#FFD700] hover:bg-[#e6c200] text-[#2D5A27] text-4xl font-display py-4 shadow-[0_6px_0_#d4a017] active:shadow-none active:translate-y-[6px]">Kembali ke Menu</button>
             </div>
           </div>
        )}
      </main>

      {activeModal === 'howTo' && (
        <Modal title="Peraturan Rimba" onClose={() => setActiveModal(null)}>
           <div className="space-y-4">
             <div className="flex items-start gap-4"><Brain className="text-[#2D5A27] mt-1" size={32} /><div><p className="font-bold text-2xl">Main Solo atau Lawan Kawan</p><p className="text-lg">Pilih mod permainan di menu utama. Masukkan nama bilik yang sama untuk bermain bersama!</p></div></div>
             <div className="flex items-start gap-4"><Zap className="text-[#FFD700] mt-1" size={32} /><div><p className="font-bold text-2xl">Tarikan Pasukan</p><p className="text-lg">Jawapan betul menarik tali ke arah anda. Siapa cepat, dia menang!</p></div></div>
           </div>
        </Modal>
      )}

      {activeModal === 'leaderboard' && (
        <Modal title="Legenda Rimba" onClose={() => setActiveModal(null)}>
          <div className="space-y-4">
            {[{ name: 'Renjer Adli', score: 2400, rank: 1 }, { name: 'Pengakap Sarah', score: 1850, rank: 2 }, { name: 'Malim Chong', score: 1200, rank: 3 }].map((player) => (
              <div key={player.rank} className="flex items-center justify-between p-4 bg-white/50 rounded-xl hand-drawn-border">
                 <div className="flex items-center gap-4"><span className={`w-10 h-10 flex items-center justify-center font-display text-2xl rounded-full ${player.rank === 1 ? 'bg-[#FFD700] text-[#2D5A27]' : 'bg-[#e8f5e9] text-[#2D5A27]'}`}>{player.rank}</span><span className="font-body text-3xl font-bold">{player.name}</span></div>
                 <span className="font-display text-3xl text-[#D44D5C]">{player.score}</span>
              </div>
            ))}
          </div>
        </Modal>
      )}
    </div>
  );
}