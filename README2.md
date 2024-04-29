//ContentView (create files first) 
import SwiftUI

struct ContentView: View {
    var body: some View {
        GamesListView()
        
        TabView {
            
            Games()
                .tabItem {
                    Image(systemName: "icon1")
                    Text("Games")
                } //tabitem
            Map()
                .tabItem {
                    Image(systemName: "icon1")
                    Text("Map")
                } //tabitemmap
            Converter_()
                .tabItem {
                    Image(systemName: "icon1")
                    Text("Converter")
                }//converter
            About()
                .tabItem {
                    Image(systemName: "icon1")
                    Text("About")
                }//about
        } // tabiew
    }
} //struct

#Preview {
    ContentView()
}

//Games 

import SwiftUI
import Foundation

struct Games: Codable, Identifiable {
    
    let id: Int
    let name: String
    let publisher: String
    let releasedDate: String 
}

//GamesViewModel 
import SwiftUI

struct GamesViewodel: ObservedObject {
    @Published var games: [Game] = []
    
    func fetchData(from url: URL) {
        URLSession.shared.dataTask(with: url) { data, response, error in
            guard let data = data, error == nil else {
                print("Error fetching data: \(error?.lcoalizedDescription ?? "Unknown error")")
                return
            }
            do {
                let decodedData = try JSONDecoder().decode([Games].self, from: data)
                DispatchQueue.main.async {
                    self.games = decodedData
                }
            } catch {
                print("Error decoding data: \(error.localizedDescription)")
            }
        }.resume()
        
    } //fetchdata
} //struct

//GamesListVIew
import SwiftUI

struct GamesListView: View {
    @ObservableObject var viewModel = GamesViewodel()

    var body: some View {
        NavigationView {
            if viewModel.games.isEmpty {
                Text("No Games available")
                    .foregroundColor(.gray)
            } else {
                List(viewModel.games, id: \.id) { game in
                    NavigationLink(destination: GameDetailedView(game: game)) {
                        VStack(alignment: .leading) {
                            Text(game.name)
                                .font(.headline)
                            Text("Publisher: \(game.publisher)")
                                .font(.subheadline)
                            Text("Release Date: \(game.releaseDate)")
                                .font(.subheadline)
                        }
                    }
                }
                .listStyle(PlainListStyle())
                .navigationTitle("Games")
            }
        }
        .onAppear{
            let url = URL(string: "INSERT API I+LINK")!
            viewModel.fetchData(from: url)
        }
    }

#Preview {
    GamesListView()
}

//GamesDetailedView
import SwiftUI

struct GameDetailedView: View {
    let game: Game
    
    var body: some View {
        VStack {
            Text(game.name)
                .font(/*@START_MENU_TOKEN@*/.title/*@END_MENU_TOKEN@*/)
            Text("Publisher: \(game.publisher)")
                .padding(.bottom)
            Text("Release Date: \(game.releaseDate)")
            Spacer()
        } //vstack
        .padding()
        .navigationTitle(game.name)
    }
}

#Preview {
    GameDetailedView()
}

//Map 
import SwiftUI
import MapKit

struct Map: View {
    
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194),
        //find coordindates for rochester to replace with
        span: MKCoordinateSpan(MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05))
    )
    var body: some View {
        Map(coordinateRegion: $region, showUserLocation: true)
            .edgesIgnoringSafeArea(/*@START_MENU_TOKEN@*/.all/*@END_MENU_TOKEN@*/)
            .overlay(
                VStack {
                    Spacer()
                    HStack {
                        Spacer()
                        Button(action: {
                            zoomToUserLocation()
                        }) {
                            Image(systemName: "location.fill")
                                .foregroundColor(.white)
                                .padding()
                                .background(Color.blue)
                                .clipShape(/*@START_MENU_TOKEN@*/Circle()/*@END_MENU_TOKEN@*/)
                        }
                        .padding()
                    }//hstack
                }
            ) //overlay
    }
    private func zoomToUserLocation() {
        let span = MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)
        region = MKCoordinateRegion(center: region.center, span: span)
    }
}

#Preview {
    Map()
}

//About 
import SwiftUI

struct About: View {
    var body: some View {
        VStack {
            Text("Tiana")
                .font(.largeTitle)
                .fontWeight(/*@START_MENU_TOKEN@*/.bold/*@END_MENU_TOKEN@*/)
                .foregroundColor(.white)
                .padding()
                .background(Color.blue)
                .cornerRadius(10)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(Color.gray.opacity(0.1))
        .edgesIgnoringSafeArea(/*@START_MENU_TOKEN@*/.all/*@END_MENU_TOKEN@*/)
    }
}

#Preview {
    About()
}

//Converter
import SwiftUI

struct Converter_: View {
    
    @State private var temperature = ""
    @State private var celsResult = ""
    @State private var fahrResult = ""
    
    var body: some View {
        VStack {
            TextField("Temperature to Convert:", text: $temperature)
                .keyboardType(.numberPad)
                .padding()
                .background(Color.white)
                .cornerRadius(10)
                .padding()
                .onTapGesture {
                    UIApplication.shared.sendAction(#selector(UIResponder.resignFirstResponder), to: nil, from: nil, for: nil)
                }
            HStack {
                Button(action: {
                    calcCelsius()
                }){
                    Text("Convert to Celsius")
                        .padding()
                        .foregroundColor(.white)
                        .background(Color.blue)
                        .cornerRadius(10)
                }
                
                Button(action: {
                    calcFahrenheit()
                }) {
                    Text("Convert to Celsius")
                        .padding()
                        .foregroundColor(.white)
                        .background(Color.blue)
                        .cornerRadius(10)
                }
                
                }
            }
        Spacer()
        Text("Celcsui: \(celsResult)C").padding()
            .foregroundColor(.black)
        
        Text("Fahrenhiet: \(fahrResult)C").padding()
            .foregroundColor(.black)
        }
        .padding()
        .background(Color.gray.opacity(0.1))
        .edgesIgnoringSafeArea(.all)
    }

private func calcCelsius() {
    guard let temp = Double(temperature) else {return}
    let celsuis = (5.0/ 9.0) * (temp - 32.0)
    celsResult = String(format: "%.2f", celsuis)
}
}

#Preview {
    Converter_()
}

