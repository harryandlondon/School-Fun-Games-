"use client"

import { useState, useEffect } from "react"
import { useRouter } from "next/navigation"
import { Button } from "@/components/ui/button"
import { Card, CardContent } from "@/components/ui/card"

interface Game {
  id: string
  title: string
  description: string
  emoji: string
  link: string
}

interface Notification {
  id: string
  gameTitle: string
  gameId: string
  reason: string
  details: string
  date: string
  dismissed: boolean
}

export default function DashboardPage() {
  const [games, setGames] = useState<Game[]>([])
  const [currentEmployee, setCurrentEmployee] = useState<string | null>(null)
  const [reportModalOpen, setReportModalOpen] = useState(false)
  const [currentGameId, setCurrentGameId] = useState<string | null>(null)
  const [reportReason, setReportReason] = useState("Inappropriate content")
  const [reportDetails, setReportDetails] = useState("")
  const router = useRouter()

  useEffect(() => {
    // Check if user is authenticated by checking session storage
    const isAuthenticated = sessionStorage.getItem("isAuthenticated")
    if (!isAuthenticated) {
      router.push("/")
      return
    }

    // Get current employee name
    const employeeData = sessionStorage.getItem("currentEmployee")
    if (employeeData) {
      setCurrentEmployee(JSON.parse(employeeData).name)
    }

    // Load games from localStorage
    const savedGames = localStorage.getItem("schoolGames")
    if (savedGames) {
      try {
        setGames(JSON.parse(savedGames))
      } catch (e) {
        console.error("Could not parse saved games", e)
      }
    }
  }, [router])

  const logout = () => {
    sessionStorage.removeItem("isAuthenticated")
    sessionStorage.removeItem("currentEmployee")
    router.push("/")
  }

  const openReportModal = (gameId: string) => {
    setCurrentGameId(gameId)
    setReportModalOpen(true)
  }

  const submitReport = () => {
    if (!currentGameId) return

    const game = games.find((g) => g.id === currentGameId)
    if (!game) return

    const notification: Notification = {
      id: Date.now().toString(),
      gameTitle: game.title,
      gameId: currentGameId,
      reason: reportReason,
      details: reportDetails,
      date: new Date().toLocaleString(),
      dismissed: false,
    }

    // Save notification to localStorage
    const savedNotifications = localStorage.getItem("gameNotifications")
    let notifications: Notification[] = []

    if (savedNotifications) {
      try {
        notifications = JSON.parse(savedNotifications)
      } catch (e) {
        console.error("Could not parse saved notifications", e)
      }
    }

    notifications.push(notification)
    localStorage.setItem("gameNotifications", JSON.stringify(notifications))

    // Reset and close modal
    setReportReason("Inappropriate content")
    setReportDetails("")
    setReportModalOpen(false)
    setCurrentGameId(null)

    alert("Thank you for your report. An administrator will review it soon.")
  }

  return (
    <div className="min-h-screen bg-[#f0f8ff]">
      <header className="bg-[#4169e1] text-white text-center p-5 border-b-8 border-[#ffd700] relative">
        <h1 className="text-4xl font-bold m-0 font-comic text-shadow">🎮 School Fun Games 🎮</h1>
        <div className="absolute top-2 right-2 flex items-center gap-2">
          {currentEmployee && <span className="text-white">Welcome, {currentEmployee}</span>}
          <Button onClick={logout} className="bg-[#ffd700] text-[#4169e1] hover:bg-[#ffd700]/80">
            Logout
          </Button>
        </div>
      </header>

      <main className="container mx-auto py-8 px-4">
        <div className="text-center mb-8">
          <h2 className="text-2xl font-bold text-[#4169e1]">Game Dashboard</h2>
          <p className="text-gray-600">Browse and play educational games</p>
        </div>

        {games.length > 0 ? (
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {games.map((game) => (
              <Card key={game.id} className="overflow-hidden transition-transform hover:-translate-y-1">
                <div className="h-32 bg-[#ffd700] flex items-center justify-center text-5xl">{game.emoji}</div>
                <CardContent className="p-4">
                  <h3 className="text-xl font-bold text-[#4169e1]">{game.title}</h3>
                  <p className="text-gray-600 mt-2">{game.description}</p>
                  <div className="flex justify-between mt-4">
                    <a href={game.link} target="_blank" rel="noopener noreferrer">
                      <Button className="bg-[#32cd32] hover:bg-[#228b22]">Play Now</Button>
                    </a>
                    <Button onClick={() => openReportModal(game.id)} className="bg-[#ff6b6b] hover:bg-[#ff4757]">
                      Report
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        ) : (
          <div className="text-center py-12 text-gray-500">
            <p className="text-xl">No games have been added yet.</p>
          </div>
        )}

        {/* Report Modal */}
        {reportModalOpen && (
          <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-lg w-full max-w-md p-6">
              <h2 className="text-xl font-bold mb-4">Report Game</h2>
              <p className="mb-4">Please tell us why you're reporting this game:</p>

              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium mb-1">Reason:</label>
                  <select
                    className="w-full p-2 border rounded"
                    value={reportReason}
                    onChange={(e) => setReportReason(e.target.value)}
                  >
                    <option value="Inappropriate content">Inappropriate content</option>
                    <option value="Not working">Game not working</option>
                    <option value="Not educational">Not educational</option>
                    <option value="Other">Other</option>
                  </select>
                </div>

                <div>
                  <label className="block text-sm font-medium mb-1">Details (optional):</label>
                  <textarea
                    className="w-full p-2 border rounded"
                    rows={3}
                    placeholder="Add any additional details here"
                    value={reportDetails}
                    onChange={(e) => setReportDetails(e.target.value)}
                  ></textarea>
                </div>

                <div className="flex justify-end gap-2">
                  <Button variant="outline" onClick={() => setReportModalOpen(false)}>
                    Cancel
                  </Button>
                  <Button onClick={submitReport} className="bg-[#32cd32]">
                    Submit Report
                  </Button>
                </div>
              </div>
            </div>
          </div>
        )}
      </main>

      <footer className="bg-[#4169e1] text-white text-center p-4 mt-8 border-t-8 border-[#ffd700]">
        <p>© {new Date().getFullYear()} School Fun Games | Created for educational purposes</p>
      </footer>
    </div>
  )
}

