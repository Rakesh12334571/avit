import React, { useState, useEffect } from 'react';
import { Card, CardContent } from "@/components/ui/card"; // Custom components for styling
import { Button } from "@/components/ui/button"; // Custom button component
import { motion } from "framer-motion";

const AviatGame = () => {
  const [multiplier, setMultiplier] = useState(1.0);
  const [isFlying, setIsFlying] = useState(false);
  const [crashed, setCrashed] = useState(false);
  const [betAmount, setBetAmount] = useState(100);
  const [balance, setBalance] = useState(1000);
  const [cashOutValue, setCashOutValue] = useState(null);

  useEffect(() => {
    const crashSocket = new WebSocket("wss://your-backend-url");

    crashSocket.onmessage = (message) => {
      if (message.data === "CRASH") {
        setCrashed(true);
        setIsFlying(false);
      }
    };

    return () => crashSocket.close();
  }, []);

  const startGame = () => {
    if (betAmount > balance) {
      alert("Insufficient balance!");
      return;
    }

    setBalance(balance - betAmount);
    setMultiplier(1.0);
    setIsFlying(true);
    setCrashed(false);
    setCashOutValue(null);

    let crashPoint = Math.random() * (5 - 1.5) + 1.5;

    const interval = setInterval(() => {
      setMultiplier((prev) => {
        if (prev >= crashPoint) {
          clearInterval(interval);
          setCrashed(true);
          setIsFlying(false);
        }
        return prev + 0.1;
      });
    }, 100);
  };

  const cashOut = () => {
    if (!isFlying || crashed) return;

    setCashOutValue(multiplier);
    setBalance(balance + betAmount * multiplier);
    setIsFlying(false);
  };

  const resetGame = () => {
    setMultiplier(1.0);
    setIsFlying(false);
    setCrashed(false);
    setCashOutValue(null);
  };

  return (
    <div className="p-6 flex flex-col items-center justify-center min-h-screen bg-gray-900 text-white">
      <Card className="w-full max-w-md mb-6">
        <CardContent className="text-center">
          <h1 className="text-2xl font-bold mb-4">AVIAT Game</h1>
          <p className="text-lg">Multiplier: <span className="font-mono text-yellow-400">{multiplier.toFixed(2)}x</span></p>
          {crashed && <p className="text-red-500 font-bold">Plane Crashed!</p>}
          {cashOutValue && <p className="text-green-500 font-bold">Cashed Out at {cashOutValue.toFixed(2)}x!</p>}
        </CardContent>
      </Card>

      <div className="flex gap-4 mb-6">
        <div className="flex flex-col items-center">
          <p>Balance: ₹{balance}</p>
          <p>Bet Amount: ₹{betAmount}</p>
          <input
            type="range"
            min="100"
            max={balance}
            step="100"
            value={betAmount}
            onChange={(e) => setBetAmount(Number(e.target.value))}
            className="mt-2"
          />
        </div>
      </div>

      <div className="flex gap-4">
        {!isFlying && !crashed && (
          <Button onClick={startGame} className="bg-blue-600 hover:bg-blue-700">
            Start Game
          </Button>
        )}
        {isFlying && (
          <Button onClick={cashOut} className="bg-green-600 hover:bg-green-700">
            Cash Out
          </Button>
        )}
        {crashed && (
          <Button onClick={resetGame} className="bg-gray-600 hover:bg-gray-700">
            Reset Game
          </Button>
        )}
      </div>

      <motion.div
        className="mt-12 bg-blue-500 w-24 h-24 rounded-full"
        animate={{ y: crashed ? 300 : isFlying ? -200 : 0 }}
        transition={{ duration: 1 }}
      />
    </div>
  );
};

const AdminPanel = () => {
  const sendCrashSignal = () => {
    const crashSocket = new WebSocket("wss://your-backend-url");
    crashSocket.onopen = () => {
      crashSocket.send("CRASH");
      crashSocket.close();
    };
  };

  return (
    <div className="p-6 flex flex-col items-center justify-center min-h-screen bg-gray-800 text-white">
      <Card className="w-full max-w-md">
        <CardContent className="text-center">
          <h1 className="text-2xl font-bold mb-4">Admin Panel</h1>
          <Button
            onClick={sendCrashSignal}
            className="bg-red-600 hover:bg-red-700 px-4 py-2 rounded"
          >
            Crash Plane
          </Button>
        </CardContent>
      </Card>
    </div>
  );
};

export default AviatGame;
export { AdminPanel };
