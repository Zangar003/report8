# report8
This code defines a function named upload that handles HTTP requests to upload and filter files.

When a user submits the form defined in the HTML code, the function checks which checkboxes were selected (rating or price). Depending on the selection, the function executes a SQL query to retrieve the files from the database and orders them based on the selected criteria.

The dbConn() function returns a database connection object to connect to the database, and the *sql.Rows object is used to store the result set of the executed SQL query.

The function then iterates over the rows of the result set and populates a []upfile slice with the retrieved file data. The upfile struct contains fields that represent the file's properties, such as ID, name, star rating, path, and price.

Finally, the function counts the number of files retrieved, and if it is greater than zero, it executes a Go template named uploadfile.html with the []upfile slice as input data. If no files were retrieved, it executes the same template but with nil as the input data.

The function uses the tmpl.ExecuteTemplate() method to execute the Go template, and the w http.ResponseWriter object to write the output HTML to the HTTP response.

At the end of the function, the database connection is closed using the db.Close() method.




The html code
<form style="margin: 20px;" method="post" action="/" >
		<h3>To Filter Choose your  features:</h3>				
		<input type="checkbox"  value="raiting" name="raiting"><label for="raiting" checked="checked" >Raiting</label><br>
		<input type="checkbox"  value="price" name="price"><label for="price" checked="checked">Price</label><br>
		<button class="submit">Submit</button>			
	</form>
  go code
  func upload(w http.ResponseWriter, r *http.Request) {
	db := dbConn()
	var selDB *sql.Rows

	if r.Method == "POST" {
		rating := r.FormValue("raiting")
		price := r.FormValue("price")

		if rating == "raiting" {
			sel, err := db.Query("SELECT * FROM `upload` ORDER BY star ASC")
			selDB = sel
			if err != nil {
				panic(err.Error())
			}
		} else if price == "price" {
			sel, err := db.Query("SELECT * FROM `upload` ORDER BY price ASC")
			selDB = sel
			if err != nil {
				panic(err.Error())
			}
		} else {
			sel, err := db.Query("SELECT * FROM upload ORDER BY id DESC")
			selDB = sel
			if err != nil {
				panic(err.Error())
			}
		}
	} else {
		sel, err := db.Query("SELECT * FROM upload ORDER BY id DESC")
		selDB = sel
		if err != nil {
			panic(err.Error())
		}
	}

	upld := upfile{}
	res := []upfile{}
	for selDB.Next() {
		var id, star, price int
		var fname, item_type, path string

		err = selDB.Scan(&id, &fname, &item_type, &star, &path, &price)
		if err != nil {
			panic(err.Error())
		}
		upld.ID = id
		upld.Fname = fname
		upld.Item_type = item_type
		upld.Star = star
		upld.Path = path
		upld.Price = price
		res = append(res, upld)

	}

	upld.Count = len(res)

	if upld.Count > 0 {
		tmpl.ExecuteTemplate(w, "uploadfile.html", res)
	} else {
		tmpl.ExecuteTemplate(w, "uploadfile.html", nil)
	}

	db.Close()

}
