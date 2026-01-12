# MisakiCore
All-in-one SMP core plugin for Misaki SMP
// MisakiCore — All‑in‑One SMP Plugin
// Target: Paper 1.21.x
// Build: Maven
// This single canvas includes ALL required files. Split into files as named.

// =========================
// FILE: pom.xml
// =========================
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>gg.misaki</groupId>
  <artifactId>MisakiCore</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <properties>
    <java.version>21</java.version>
  </properties>

  <repositories>
    <repository>
      <id>papermc</id>
      <url>https://repo.papermc.io/repository/maven-public/</url>
    </repository>
  </repositories>

  <dependencies>
    <dependency>
      <groupId>io.papermc.paper</groupId>
      <artifactId>paper-api</artifactId>
      <version>1.21.1-R0.1-SNAPSHOT</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>

// =========================
// FILE: src/main/resources/plugin.yml
// =========================
name: MisakiCore
version: 1.0.0
main: gg.misaki.core.MisakiCore
api-version: '1.21'
author: Misaki SMP
commands:
  skills:
  leaderboard:
  villagerpickup:
permissions:
  misaki.admin:
    default: op
  misaki.use:
    default: true

// =========================
// FILE: src/main/java/gg/misaki/core/MisakiCore.java
// =========================
package gg.misaki.core;

import org.bukkit.*;
import org.bukkit.command.*;
import org.bukkit.entity.*;
import org.bukkit.event.*;
import org.bukkit.event.entity.*;
import org.bukkit.event.player.*;
import org.bukkit.inventory.*;
import org.bukkit.inventory.meta.*;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.*;

public class MisakiCore extends JavaPlugin implements Listener {

    private final Map<UUID, Integer> combatXP = new HashMap<>();
    private final Map<UUID, Integer> kills = new HashMap<>();

    @Override
    public void onEnable() {
        saveDefaultConfig();
        Bukkit.getPluginManager().registerEvents(this, this);
        getLogger().info("MisakiCore enabled for Paper 1.21.x");
    }

    // =========================
    // SKILLS + KILL TRACKING
    // =========================
    @EventHandler
    public void onKill(PlayerDeathEvent e) {
        if (e.getEntity().getKiller() == null) return;
        Player p = e.getEntity().getKiller();
        kills.put(p.getUniqueId(), kills.getOrDefault(p.getUniqueId(), 0) + 1);
        combatXP.put(p.getUniqueId(), combatXP.getOrDefault(p.getUniqueId(), 0) + 10);

        // Kill effect
        p.getWorld().strikeLightningEffect(e.getEntity().getLocation());
        p.sendMessage(ChatColor.RED + "Kill confirmed! +10 Combat XP");
    }

    // =========================
    // BASIC ANTI‑CHEAT (REACH CHECK)
    // =========================
    @EventHandler
    public void onHit(EntityDamageByEntityEvent e) {
        if (!(e.getDamager() instanceof Player p) || !(e.getEntity() instanceof Player t)) return;
        if (p.getLocation().distance(t.getLocation()) > 4.2) {
            e.setCancelled(true);
            p.sendMessage(ChatColor.RED + "[MisakiAC] Suspicious reach detected.");
        }
    }

    // =========================
    // VILLAGER PICKUP
    // =========================
    @EventHandler
    public void onInteract(PlayerInteractEntityEvent e) {
        if (!(e.getRightClicked() instanceof Villager v)) return;
        Player p = e.getPlayer();
        ItemStack villagerItem = new ItemStack(Material.EMERALD);
        ItemMeta meta = villagerItem.getItemMeta();
        meta.setDisplayName(ChatColor.GREEN + "Captured Villager");
        villagerItem.setItemMeta(meta);
        p.getInventory().addItem(villagerItem);
        v.remove();
        p.sendMessage(ChatColor.GREEN + "Villager stored in inventory.");
    }

    // =========================
    // COMMANDS
    // =========================
    @Override
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        if (!(s instanceof Player p)) return true;

        if (c.getName().equalsIgnoreCase("skills")) {
            int xp = combatXP.getOrDefault(p.getUniqueId(), 0);
            p.sendMessage(ChatColor.AQUA + "Combat XP: " + xp);
            return true;
        }

        if (c.getName().equalsIgnoreCase("leaderboard")) {
            p.sendMessage(ChatColor.GOLD + "Top Kills (local):");
            kills.entrySet().stream()
                    .sorted((a1, a2) -> a2.getValue() - a1.getValue())
                    .limit(5)
                    .forEach(e -> p.sendMessage(Bukkit.getOfflinePlayer(e.getKey()).getName() + ": " + e.getValue()));
            return true;
        }
        return false;
    }
}

// =========================
// BUILD INSTRUCTIONS
// =========================
// 1. Install Java 21
// 2. Save files exactly as named
// 3. Run: mvn package
// 4. Upload target/MisakiCore-1.0.0.jar to /plugins
// 5. Restart server
