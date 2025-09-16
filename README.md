package main

import (
	"encoding/json"
	"log"
	"net/http"
	"strconv"
)

type Todo struct {
	ID     int    `json:"id"`
	Task   string `json:"task"`
	Done   bool   `json:"done"`
}

var todos []Todo
var idCounter = 1

func getTodos(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(todos)
}

func addTodo(w http.ResponseWriter, r *http.Request) {
	var todo Todo
	_ = json.NewDecoder(r.Body).Decode(&todo)
	todo.ID = idCounter
	idCounter++
	todos = append(todos, todo)
	json.NewEncoder(w).Encode(todo)
}

func toggleTodo(w http.ResponseWriter, r *http.Request) {
	id, _ := strconv.Atoi(r.URL.Query().Get("id"))
	for i := range todos {
		if todos[i].ID == id {
			todos[i].Done = !todos[i].Done
			json.NewEncoder(w).Encode(todos[i])
			return
		}
	}
	http.NotFound(w, r)
}

func main() {
	http.HandleFunc("/todos", getTodos)
	http.HandleFunc("/add", addTodo)
	http.HandleFunc("/toggle", toggleTodo)

	log.Println("✅ Todo API running at http://localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
