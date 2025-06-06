import React, { useEffect, useState } from 'react';

export default function CarRacingApp() {
  const [carPosition, setCarPosition] = useState(200);
  const [obstacles, setObstacles] = useState([
    { top: 0, left: Math.random() * 400 }
  ]);
  const [score, setScore] = useState(0);
  const [highScore, setHighScore] = useState(
    parseInt(localStorage.getItem('highScore')) || 0
  );
  const [gameOver, setGameOver] = useState(false);
  const [speed, setSpeed] = useState(10);
  const [gameStarted, setGameStarted] = useState(false);

  useEffect(() => {
    const handleKeyDown = (e) => {
      if (!gameStarted || gameOver) return;
      if (e.key === 'ArrowLeft' && carPosition > 0) {
        setCarPosition((prev) => prev - 20);
      } else if (e.key === 'ArrowRight' && carPosition < 380) {
        setCarPosition((prev) => prev + 20);
      }
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [carPosition, gameOver, gameStarted]);

  useEffect(() => {
    if (!gameStarted || gameOver) return;
    const interval = setInterval(() => {
      setObstacles((prevObstacles) => {
        const newObstacles = prevObstacles.map((obs) => {
          const newTop = obs.top + speed;
          if (newTop > 450 && Math.abs(obs.left - carPosition) < 50) {
            new Audio('/crash.mp3').play();
            setGameOver(true);
            return obs;
          }
          return { ...obs, top: newTop };
        });

        const filtered = newObstacles.filter((obs) => obs.top <= 500);
        const passed = newObstacles.length - filtered.length;
        if (passed > 0) {
          new Audio('/point.mp3').play();
          setScore((s) => s + passed);
        }

        while (filtered.length < 3) {
          filtered.push({ top: 0, left: Math.random() * 400 });
        }

        return filtered;
      });

      setSpeed((s) => Math.min(s + 0.05, 30));
    }, 100);

    return () => clearInterval(interval);
  }, [carPosition, gameOver, speed, gameStarted]);

  useEffect(() => {
    if (gameOver) {
      if (score > highScore) {
        setHighScore(score);
        localStorage.setItem('highScore', score);
      }
    }
  }, [gameOver]);

  const handleTouch = (direction) => {
    if (!gameStarted || gameOver) return;
    if (direction === 'left' && carPosition > 0) {
      setCarPosition((prev) => prev - 20);
    } else if (direction === 'right' && carPosition < 380) {
      setCarPosition((prev) => prev + 20);
    }
  };

  const startGame = () => {
    setCarPosition(200);
    setObstacles([{ top: 0, left: Math.random() * 400 }]);
    setScore(0);
    setSpeed(10);
    setGameOver(false);
    setGameStarted(true);
  };

  return (
    <div className="w-[500px] h-[500px] bg-gray-800 relative overflow-hidden border-2 border-white mx-auto mt-10">
      <img
        src="/car.png"
        alt="car"
        className="absolute bottom-2 w-12 h-20 object-contain"
        style={{ left: carPosition }}
      />
      {obstacles.map((obs, index) => (
        <div
          key={index}
          className="absolute w-12 h-20 bg-red-500 rounded"
          style={{ top: obs.top, left: obs.left }}
        />
      ))}
      <div className="absolute top-2 left-2 text-white text-lg">Score: {score}</div>
      <div className="absolute top-2 right-2 text-yellow-300 text-lg">High Score: {highScore}</div>

      {!gameStarted && (
        <div className="absolute inset-0 flex flex-col items-center justify-center bg-black bg-opacity-80 text-white">
          <h1 className="text-3xl mb-4">Car Racing Game</h1>
          <button
            className="bg-green-500 px-6 py-3 rounded text-lg"
            onClick={startGame}
          >
            Start Game
          </button>
        </div>
      )}

      {gameOver && (
        <div className="absolute inset-0 flex flex-col items-center justify-center bg-black bg-opacity-75 text-white text-2xl">
          <p>Game Over! Final Score: {score}</p>
          <button
            className="mt-4 px-4 py-2 bg-blue-500 rounded text-lg"
            onClick={startGame}
          >
            Restart
          </button>
        </div>
      )}

      <div className="absolute bottom-2 left-2 flex gap-2 sm:hidden">
        <button
          className="bg-white text-black px-4 py-2 rounded"
          onClick={() => handleTouch('left')}
        >
          ◀
        </button>
        <button
          className="bg-white text-black px-4 py-2 rounded"
          onClick={() => handleTouch('right')}
        >
          ▶
        </button>
      </div>
    </div>
  );
}
