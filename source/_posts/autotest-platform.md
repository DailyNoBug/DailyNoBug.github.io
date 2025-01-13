---
title: autotest-platform
date: 2024-06-21 02:08:57
tags:
---

# Autotest Platform

An ssh host management interface. On the left is the ssh connection card, and on the right is the ssh terminal. Click different cards to enter different terminals. The circular plus sign in the lower left can add ssh host cards. There is a hidden - sign on the right side of the card, which appears when the mouse hovers over the card. Click the - sign to delete the card, and the corresponding ssh terminal will also be deleted.



There is a hidden area on the right with a < sign. Hovering the mouse over it will pop up a page on the right 1/3 area. The upper part of the page is the user's fixed command card. You can click the + sign to create a new command. Hovering the mouse over a command will pop up a - sign. Click to delete the command. Click the card to execute the command in the left terminal. At the bottom of the page is the history command card. Click the card to execute the command in the left terminal. Hovering the mouse will pop up a - sign. Click to delete the command.





```jsx
/**
 * v0 by Vercel.
 * @see https://v0.dev/t/Oz8N7wjr9r6
 * Documentation: https://v0.dev/docs#integrating-generated-code-into-your-nextjs-app
 */
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Card, CardContent } from "@/components/ui/card"
import { Avatar, AvatarImage, AvatarFallback } from "@/components/ui/avatar"
import { Input } from "@/components/ui/input"
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription } from "@/components/ui/dialog"
import { Label } from "@/components/ui/label"

export default function Component() {
  const [showModal, setShowModal] = useState(false)
  const [newHost, setNewHost] = useState({
    name: "",
    ip: "",
  })
  const handleInputChange = (e) => {
    setNewHost({
      ...newHost,
      [e.target.name]: e.target.value,
    })
  }
  const handleAddHost = () => {
    console.log("New host:", newHost)
    setShowModal(false)
  }
  const handleDeleteHost = (host) => {
    console.log("Deleting host:", host)
  }
  return (
    <div className="grid grid-cols-[300px_1fr] h-screen w-full">
      <div className="bg-muted/40 border-r p-4 flex flex-col gap-4 overflow-auto">
        <div className="flex items-center justify-between">
          <h2 className="text-lg font-semibold">SSH Hosts</h2>
          <Button
            variant="ghost"
            size="icon"
            className="text-muted-foreground hover:bg-muted"
            onClick={() => setShowModal(true)}
          >
            <CirclePlusIcon className="w-6 h-6" />
            <span className="sr-only">Add new SSH host</span>
          </Button>
        </div>
        <div className="grid gap-4">
          <Card className="relative group">
            <CardContent className="flex items-center justify-between">
              <div className="flex items-center gap-3">
                <Avatar>
                  <AvatarImage src="/placeholder-user.jpg" />
                  <AvatarFallback>HO</AvatarFallback>
                </Avatar>
                <div>
                  <div className="font-medium">Production Server</div>
                  <div className="text-sm text-muted-foreground">192.168.1.100</div>
                </div>
              </div>
              <Button
                variant="ghost"
                size="icon"
                className="opacity-0 group-hover:opacity-100 text-muted-foreground hover:bg-muted"
                onClick={() => handleDeleteHost("Production Server")}
              >
                <CircleMinusIcon className="w-5 h-5" />
                <span className="sr-only">Delete SSH host</span>
              </Button>
            </CardContent>
          </Card>
          <Card className="relative group">
            <CardContent className="flex items-center justify-between">
              <div className="flex items-center gap-3">
                <Avatar>
                  <AvatarImage src="/placeholder-user.jpg" />
                  <AvatarFallback>DE</AvatarFallback>
                </Avatar>
                <div>
                  <div className="font-medium">Development Server</div>
                  <div className="text-sm text-muted-foreground">192.168.1.101</div>
                </div>
              </div>
              <Button
                variant="ghost"
                size="icon"
                className="opacity-0 group-hover:opacity-100 text-muted-foreground hover:bg-muted"
                onClick={() => handleDeleteHost("Development Server")}
              >
                <CircleMinusIcon className="w-5 h-5" />
                <span className="sr-only">Delete SSH host</span>
              </Button>
            </CardContent>
          </Card>
          <Card className="relative group">
            <CardContent className="flex items-center justify-between">
              <div className="flex items-center gap-3">
                <Avatar>
                  <AvatarImage src="/placeholder-user.jpg" />
                  <AvatarFallback>ST</AvatarFallback>
                </Avatar>
                <div>
                  <div className="font-medium">Staging Server</div>
                  <div className="text-sm text-muted-foreground">192.168.1.102</div>
                </div>
              </div>
              <Button
                variant="ghost"
                size="icon"
                className="opacity-0 group-hover:opacity-100 text-muted-foreground hover:bg-muted"
                onClick={() => handleDeleteHost("Staging Server")}
              >
                <CircleMinusIcon className="w-5 h-5" />
                <span className="sr-only">Delete SSH host</span>
              </Button>
            </CardContent>
          </Card>
        </div>
      </div>
      <div className="flex flex-col">
        <div className="bg-muted/40 border-b p-4 flex items-center justify-between">
          <h2 className="text-lg font-semibold">SSH Terminal</h2>
          <div className="flex items-center gap-2">
            <Button variant="ghost" size="icon" className="text-muted-foreground hover:bg-muted">
              <SettingsIcon className="w-5 h-5" />
              <span className="sr-only">Settings</span>
            </Button>
            <Button variant="ghost" size="icon" className="text-muted-foreground hover:bg-muted">
              <MaximizeIcon className="w-5 h-5" />
              <span className="sr-only">Maximize</span>
            </Button>
          </div>
        </div>
        <div className="flex-1 p-4 overflow-auto">
          <div className="h-full bg-background rounded-lg border shadow-sm flex flex-col">
            <div className="flex-1 p-4">
              <pre className="font-mono text-sm whitespace-pre-wrap break-all">{`Welcome to the SSH terminal!
You are connected to the Production Server.
Type 'help' to see available commands.
`}</pre>
            </div>
            <div className="border-t p-4 flex items-center gap-2">
              <Input className="flex-1" placeholder="Type your command..." />
              <Button variant="ghost" size="icon" className="text-muted-foreground hover:bg-muted">
                <SendIcon className="w-5 h-5" />
                <span className="sr-only">Send command</span>
              </Button>
            </div>
          </div>
        </div>
      </div>
      <Dialog open={showModal} onOpenChange={setShowModal}>
        <DialogContent className="p-6 bg-background rounded-lg shadow-lg max-w-md w-full">
          <DialogHeader>
            <DialogTitle>Add New SSH Host</DialogTitle>
            <DialogDescription>Enter the details of the new SSH host you want to add.</DialogDescription>
          </DialogHeader>
          <form onSubmit={handleAddHost}>
            <div className="grid gap-4">
              <div className="space-y-2">
                <Label htmlFor="name">Name</Label>
                <Input
                  id="name"
                  name="name"
                  value={newHost.name}
                  onChange={handleInputChange}
                  placeholder="Enter host name"
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="ip">IP Address</Label>
                <Input
                  id="ip"
                  name="ip"
                  value={newHost.ip}
                  onChange={handleInputChange}
                  placeholder="Enter host IP address"
                />
              </div>
              <div className="flex justify-end gap-2">
                <Button variant="ghost" onClick={() => setShowModal(false)}>
                  Cancel
                </Button>
                <Button type="submit">Add Host</Button>
              </div>
            </div>
          </form>
        </DialogContent>
      </Dialog>
    </div>
  )
}

function CircleMinusIcon(props) {
  return (
    <svg
      {...props}
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      <circle cx="12" cy="12" r="10" />
      <path d="M8 12h8" />
    </svg>
  )
}


function CirclePlusIcon(props) {
  return (
    <svg
      {...props}
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      <circle cx="12" cy="12" r="10" />
      <path d="M8 12h8" />
      <path d="M12 8v8" />
    </svg>
  )
}


function MaximizeIcon(props) {
  return (
    <svg
      {...props}
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      <path d="M8 3H5a2 2 0 0 0-2 2v3" />
      <path d="M21 8V5a2 2 0 0 0-2-2h-3" />
      <path d="M3 16v3a2 2 0 0 0 2 2h3" />
      <path d="M16 21h3a2 2 0 0 0 2-2v-3" />
    </svg>
  )
}


function SendIcon(props) {
  return (
    <svg
      {...props}
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      <path d="m22 2-7 20-4-9-9-4Z" />
      <path d="M22 2 11 13" />
    </svg>
  )
}


function SettingsIcon(props) {
  return (
    <svg
      {...props}
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
    >
      <path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z" />
      <circle cx="12" cy="12" r="3" />
    </svg>
  )
}
```

```css
:root {
  --radius: 0.625rem;
}

body {
  font-family: var(--font-inter), sans-serif;
}

h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-inter), sans-serif;
}
```

```jsx
// This is the root layout component for your Next.js app.
// Learn more: https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required

import { Inter } from 'next/font/google'
import './styles.css'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
})

export default function Layout({ children }) {
  return (
    <html lang="en">
      <body className={inter.variable}>
        {children}
      </body>
    </html>
  )
}
```

