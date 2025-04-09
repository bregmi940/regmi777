gorkhali/
├── app/
│   ├── api/
│   │   └── verify/
│   │       └── route.ts
import { type NextRequest, NextResponse } from "next/server"
import { verifyCloudProof, type IVerifyResponse } from "@worldcoin/minikit-js"

// Define the success payload type
type MiniAppVerifyActionSuccessPayload = {
  status: "success"
  proof: string
  merkle_root: string
  nullifier_hash: string
  verification_level: string
  version: number
}

// Define the error payload type for completeness
type MiniAppVerifyActionErrorPayload = {
  status: "error"
  code: string
  detail: string
}

interface IRequestPayload {
  payload: MiniAppVerifyActionSuccessPayload
  action: string
  signal: string | undefined
  user_id?: string // Optional user ID for database integration
}

export async function POST(req: NextRequest) {
  const { payload, action, signal, user_id } = (await req.json()) as IRequestPayload

  // Use the environment variable for the app ID
  const app_id = process.env.WORLDCOIN_APP_ID as `app_${string}`

  try {
    // Verify the proof with WorldCoin
    const verifyRes = (await verifyCloudProof(payload, app_id, action, signal)) as IVerifyResponse

    if (verifyRes.success) {
      // If verification is successful, store in localStorage
      // In a real app, you would store this in a database
      if (typeof localStorage !== "undefined" && user_id) {
        localStorage.setItem(`verified_${user_id}`, "true")
      }

      return NextResponse.json({
        success: true,
        verifyRes,
        message: "Verification successful",
        status: 200,
      })
    } else {
      // Handle verification failure
      return NextResponse.json({
        success: false,
        message: verifyRes.error || "Verification failed",
        verifyRes,
        status: 400,
      })
    }
  } catch (error) {
    console.error("Verification error:", error)
    return NextResponse.json({
      success: false,
      message: "An error occurred during verification",
      status: 500,
    })
  }
}
│   ├── globals.css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 255, 255, 255;
  --background-start: 93, 63, 211;
  --background-end: 67, 56, 202;
}

body {
  color: rgb(var(--foreground-rgb));
  background: linear-gradient(to bottom, rgb(var(--background-start)), rgb(var(--background-end)));
  min-height: 100vh;
}

.tap-animation {
  transform: scale(0.9);
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation: spin 3s linear infinite;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes glow {
  0%,
  100% {
    box-shadow: 0 0 10px rgba(103, 232, 249, 0.3);
  }
  50% {
    box-shadow: 0 0 20px rgba(103, 232, 249, 0.6);
  }
}

.animate-glow {
  animation: glow 2s ease-in-out infinite;
}
│   ├── layout.tsx
import type React from "react"
import type { Metadata } from "next"
import { Inter } from 'next/font/google'
import "./globals.css"
import MiniKitProvider from "@/components/minikit-provider"
import { VerificationProvider } from "@/contexts/verification-context"

const inter = Inter({ subsets: ["latin"] })

export const metadata: Metadata = {
  title: "Gorkhali - Mine GKR",
  description: "Mine GKR by playing games and inviting friends",
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <MiniKitProvider>
          <VerificationProvider>{children}</VerificationProvider>
        </MiniKitProvider>
      </body>
    </html>
  )
}
│   └── page.tsx
"use client"

import { useState, useEffect } from "react"
import Header from "@/components/header"
import HomeSection from "@/components/home-section"
import MenuSection from "@/components/menu-section"
import GameSection from "@/components/game-section"
import MiningSection from "@/components/mining-section"
import ReferralSection from "@/components/referral-section"
import StakingSection from "@/components/staking-section"
import ClaimGKRSection from "@/components/claim-gkr-section"
import SubscribeSection from "@/components/subscribe-section"
import FundRaisingSection from "@/components/fund-raising-section"
import BuyPlaySection from "@/components/buy-play-section"
import DashboardSection from "@/components/dashboard-section"
import VerificationGate from "@/components/verification-gate"
import { useVerification } from "@/contexts/verification-context"

export default function Home() {
  const [currentSection, setCurrentSection] = useState("home")
  const [gkrBalance, setGkrBalance] = useState(0)
  const { isVerified } = useVerification()

  // Load GKR balance from localStorage
  useEffect(() => {
    const storedBalance = localStorage.getItem("gkr_balance")
    if (storedBalance) {
      setGkrBalance(Number.parseInt(storedBalance, 10))
    }
  }, [])

  // Save GKR balance to localStorage when it changes
  useEffect(() => {
    localStorage.setItem("gkr_balance", gkrBalance.toString())
  }, [gkrBalance])

  // Handle adding GKR
  const handleAddGkr = (amount: number) => {
    setGkrBalance((prev) => prev + amount)
  }

  return (
    <VerificationGate>
      <main className="flex min-h-screen flex-col items-center p-6 max-w-md mx-auto">
        <Header />

        {currentSection === "home" && (
          <HomeSection
            gkrBalance={gkrBalance}
            onStartMining={() => setCurrentSection("mining")}
            onShowGame={() => setCurrentSection("game")}
            onShowReferral={() => setCurrentSection("referral")}
            onShowStaking={() => setCurrentSection("staking")}
            onShowMenu={() => setCurrentSection("menu")}
          />
        )}

        {currentSection === "menu" && (
          <MenuSection
            onGoBack={() => setCurrentSection("home")}
            onClaimGKR={() => setCurrentSection("claim-gkr")}
            onSubscribe={() => setCurrentSection("subscribe")}
            onFundRaising={() => setCurrentSection("fund-raising")}
            onPremiumTapGame={() => setCurrentSection("buy-play")}
            onDashboard={() => setCurrentSection("dashboard")}
          />
        )}

        {currentSection === "game" && <GameSection onGoBack={() => setCurrentSection("home")} addGkr={handleAddGkr} />}

        {currentSection === "mining" && (
          <MiningSection onGoBack={() => setCurrentSection("home")} addGkr={handleAddGkr} />
        )}

        {currentSection === "referral" && <ReferralSection onGoBack={() => setCurrentSection("home")} />}

        {currentSection === "staking" && (
          <StakingSection onGoBack={() => setCurrentSection("home")} gkrBalance={gkrBalance} addGkr={handleAddGkr} />
        )}

        {currentSection === "claim-gkr" && (
          <ClaimGKRSection onGoBack={() => setCurrentSection("menu")} gkrBalance={gkrBalance} addGkr={handleAddGkr} />
        )}

        {currentSection === "subscribe" && <SubscribeSection onGoBack={() => setCurrentSection("menu")} />}

        {currentSection === "fund-raising" && (
          <FundRaisingSection
            onGoBack={() => setCurrentSection("menu")}
            gkrBalance={gkrBalance}
            addGkr={handleAddGkr}
          />
        )}

        {currentSection === "buy-play" && (
          <BuyPlaySection onGoBack={() => setCurrentSection("menu")} gkrBalance={gkrBalance} addGkr={handleAddGkr} />
        )}

        {currentSection === "dashboard" && <DashboardSection onGoBack={() => setCurrentSection("menu")} />}

        <footer className="mt-10 text-center text-sm text-white/80">© 2025 All Rights Reserved</footer>
      </main>
    </VerificationGate>
  )
}
├── components/
│   ├── buy-play-section.tsx
│   ├── claim-gkr-section.tsx
│   ├── dashboard-section.tsx
│   ├── fund-raising-section.tsx
│   ├── game-section.tsx
│   ├── header.tsx
│   ├── home-section.tsx
│   ├── menu-section.tsx
│   ├── minikit-provider.tsx
"use client"

import { type ReactNode, useEffect } from "react"

export default function MiniKitProvider({ children }: { children: ReactNode }) {
  useEffect(() => {
    try {
      // Check if MiniKit is available in the window object
      if (typeof window !== "undefined" && window.MiniKit && typeof window.MiniKit.install === "function") {
        // Install MiniKit with your app ID
        const APP_ID = process.env.NEXT_PUBLIC_WORLDCOIN_APP_ID || "app_staging_9e4f5a2e9c6c3c1e9e4f5a2e9c6c3c1e"
        window.MiniKit.install(APP_ID)
        console.log("MiniKit installed successfully")
      } else {
        console.log("MiniKit is not available in this environment")
      }
    } catch (error) {
      console.error("Error installing MiniKit:", error)
    }
  }, [])

  return <>{children}</>
}
│   ├── mining-section.tsx
│   ├── referral-section.tsx
│   ├── staking-section.tsx
│   ├── subscribe-section.tsx
│   ├── ui/
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── input.tsx
│   │   └── tabs.tsx
│   └── verification-gate.tsx
"use client"

import type { ReactNode } from "react"
import { useVerification } from "@/contexts/verification-context"
import { Button } from "@/components/ui/button"
import { useEffect, useState } from "react"

interface VerificationGateProps {
  children: ReactNode
}

export default function VerificationGate({ children }: VerificationGateProps) {
  const { isVerified, isVerifying, verificationError, handleVerify } = useVerification()
  const [isWorldAppEnvironment, setIsWorldAppEnvironment] = useState(true)

  useEffect(() => {
    // Check if we're in a browser and if MiniKit is available
    if (typeof window !== "undefined") {
      setIsWorldAppEnvironment(!!window.MiniKit && typeof window.MiniKit.isInstalled === "function")
    }
  }, [])

  // If user is verified, show the children
  if (isVerified) {
    return <>{children}</>
  }

  // Show verification screen
  return (
    <div className="min-h-screen flex flex-col items-center justify-center p-6 bg-gradient-to-b from-[rgb(93,63,211)] to-[rgb(67,56,202)]">
      <div className="w-full max-w-md flex flex-col items-center text-center">
        <div className="relative h-24 w-24 mx-auto mb-6">
          <img src="/logo.jpg" alt="Gorkhali Logo" className="object-contain rounded-full" />
        </div>

        <h1 className="text-2xl font-bold mb-2">Welcome to Gorkhali</h1>
        <p className="mb-8">Verify your identity with World ID to access the app</p>

        <div className="bg-white/10 border-2 border-cyan-300/50 shadow-[0_0_15px_rgba(103,232,249,0.3)] rounded-lg p-6 mb-6 w-full">
          <img src="https://worldcoin.org/icons/logo-gradient.svg" alt="World ID" className="h-12 mx-auto mb-4" />
          <p className="font-medium mb-4">Prove you're human with World ID</p>
          <p className="text-sm mb-6">
            Verify once to access all features of the Gorkhali app and earn bonus GKR tokens.
          </p>

          {!isWorldAppEnvironment ? (
            <div className="bg-yellow-500/20 border border-yellow-500/50 text-white p-4 rounded text-center mb-4">
              <p className="font-medium mb-2">World App Environment Not Detected</p>
              <p className="text-sm">
                This app is designed to run inside World App. Please open this app in World App for full functionality.
              </p>
            </div>
          ) : (
            <Button onClick={handleVerify} disabled={isVerifying} variant="gold" className="w-full rounded-xl">
              {isVerifying ? (
                <>
                  <div className="animate-spin mr-2 h-4 w-4 border-2 border-white rounded-full border-t-transparent"></div>
                  Verifying...
                </>
              ) : (
                <>Verify with World ID</>
              )}
            </Button>
          )}

          {verificationError && <p className="mt-4 text-red-300 text-sm">{verificationError}</p>}
        </div>

        <p className="text-xs text-white/70">
          Your privacy is protected. World ID verification proves you're human without revealing personal information.
        </p>
      </div>
    </div>
  )
}
├── contexts/
│   └── verification-context.tsx
"use client"

import { createContext, useContext, useState, useEffect, type ReactNode } from "react"
import { type VerifyCommandInput, VerificationLevel, type ISuccessResult } from "@worldcoin/minikit-js"

interface VerificationContextType {
  isVerified: boolean
  isVerifying: boolean
  verificationError: string | null
  handleVerify: () => Promise<void>
}

const VerificationContext = createContext<VerificationContextType | undefined>(undefined)

export function VerificationProvider({ children }: { children: ReactNode }) {
  const [isVerified, setIsVerified] = useState(false)
  const [isVerifying, setIsVerifying] = useState(false)
  const [verificationError, setVerificationError] = useState<string | null>(null)
  const [userId, setUserId] = useState<string | null>(null)

  // Get the app ID from environment variables
  const APP_ID = process.env.NEXT_PUBLIC_WORLDCOIN_APP_ID || "app_staging_9e4f5a2e9c6c3c1e9e4f5a2e9c6c3c1e"
  const ACTION_ID = "gorkhali-app-verification"

  // Generate a unique user ID if not already set
  useEffect(() => {
    if (!userId) {
      // Generate a random user ID for this session
      const generatedUserId = `user_${Math.random().toString(36).substring(2, 15)}`
      setUserId(generatedUserId)

      // Store in localStorage for persistence
      localStorage.setItem("gorkhali_user_id", generatedUserId)
    }
  }, [userId])

  // Check if user is already verified on mount
  useEffect(() => {
    const checkVerification = async () => {
      try {
        // Try to get stored user ID from localStorage
        const storedUserId = localStorage.getItem("gorkhali_user_id")
        if (storedUserId) {
          setUserId(storedUserId)

          // Check localStorage for verification status
          const verified = localStorage.getItem(`verified_${storedUserId}`) === "true"
          if (verified) {
            setIsVerified(true)
            return
          }
        }

        // If not verified in localStorage, check MiniKit as fallback
        if (typeof window !== "undefined" && window.MiniKit && typeof window.MiniKit.isInstalled === "function") {
          if (window.MiniKit.isInstalled()) {
            const isVerified = await window.MiniKit.isVerified()
            setIsVerified(isVerified)
          } else {
            console.log("MiniKit is not installed. Make sure you're running the application inside of World App")
          }
        } else {
          console.log("MiniKit is not available in this environment")
        }
      } catch (error) {
        console.error("Error checking verification:", error)
      }
    }

    checkVerification()
  }, [])

  const handleVerify = async () => {
    setIsVerifying(true)
    setVerificationError(null)

    try {
      // Check if MiniKit is available and installed
      if (typeof window === "undefined" || !window.MiniKit || typeof window.MiniKit.isInstalled !== "function") {
        setVerificationError("World App integration is not available in this environment")
        setIsVerifying(false)
        return
      }

      if (!window.MiniKit.isInstalled()) {
        setVerificationError("World App is not installed. Please open this app inside World App.")
        setIsVerifying(false)
        return
      }

      const verifyPayload: VerifyCommandInput = {
        action: ACTION_ID, // Your action ID from the Developer Portal
        signal: "0x" + Math.random().toString(16).substring(2, 10), // Random signal for demo
        verification_level: VerificationLevel.Orb, // Orb | Device
      }

      // World App will open a drawer prompting the user to confirm the operation
      const { finalPayload } = await window.MiniKit.commandsAsync.verify(verifyPayload)

      if (finalPayload.status === "error") {
        console.log("Error payload", finalPayload)
        setVerificationError("Verification failed")
        setIsVerifying(false)
        return
      }

      // Verify the proof in the backend
      const verifyResponse = await fetch("/api/verify", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          payload: finalPayload as ISuccessResult,
          action: ACTION_ID,
          signal: verifyPayload.signal,
          app_id: APP_ID,
          user_id: userId, // Pass the user ID to store verification in database
        }),
      })

      const verifyResponseJson = await verifyResponse.json()

      if (verifyResponseJson.success) {
        setIsVerified(true)
        // Store verification status in localStorage
        if (userId) {
          localStorage.setItem(`verified_${userId}`, "true")
        }
        console.log("Verification success!")
      } else {
        setVerificationError(verifyResponseJson.message || "Verification failed")
      }
    } catch (error) {
      console.error("Verification error:", error)
      setVerificationError("An error occurred during verification")
    } finally {
      setIsVerifying(false)
    }
  }

  return (
    <VerificationContext.Provider
      value={{
        isVerified,
        isVerifying,
        verificationError,
        handleVerify,
      }}
    >
      {children}
    </VerificationContext.Provider>
  )
}

export function useVerification() {
  const context = useContext(VerificationContext)
  if (context === undefined) {
    throw new Error("useVerification must be used within a VerificationProvider")
  }
  return context
}
├── lib/
│   └── utils.ts
├── public/
│   └── logo.jpg
├── types/
│   └── minikit.d.ts
interface MiniKitInstance {
  install: (appId?: string) => void
  verify: () => Promise<boolean>
  isVerified: () => Promise<boolean>
  isInstalled: () => boolean
  appId?: string
  commandsAsync: {
    verify: (payload: any) => Promise<any>
  }
}

declare global {
  interface Window {
    MiniKit?: MiniKitInstance
  }
}

export {}
├── .env.local
WORLDCOIN_APP_ID=app_029719937c6bd51a224c68e08c385cba
NEXT_PUBLIC_WORLDCOIN_APP_ID=app_029719937c6bd51a224c68e08c385cba
├── next.config.js
├── package.json
{
  "name": "gorkhali-mini-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@radix-ui/react-slot": "^1.0.2",
    "@radix-ui/react-tabs": "^1.0.4",
    "@worldcoin/minikit-js": "^0.1.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.0.0",
    "lucide-react": "^0.294.0",
    "next": "14.0.4",
    "react": "^18",
    "react-dom": "^18",
    "tailwind-merge": "^2.1.0"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "autoprefixer": "^10.0.1",
    "eslint": "^8",
    "eslint-config-next": "14.0.4",
    "postcss": "^8",
    "tailwindcss": "^3.3.0",
    "typescript": "^5"
  }
}
├── tailwind.config.ts
import type { Config } from "tailwindcss"

const config: Config = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
  plugins: [],
}

export default config
└── tsconfig.json
