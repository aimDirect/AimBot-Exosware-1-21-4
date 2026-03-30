package ru.levin.modules.combat;

import net.minecraft.entity.Entity;
import net.minecraft.entity.LivingEntity;
import net.minecraft.entity.decoration.ArmorStandEntity;
import net.minecraft.entity.mob.Monster;
import net.minecraft.entity.passive.AnimalEntity;
import net.minecraft.entity.passive.VillagerEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.util.math.MathHelper;
import net.minecraft.util.math.Vec3d;
import ru.levin.events.Event;
import ru.levin.events.impl.EventUpdate;
import ru.levin.manager.Manager;
import ru.levin.modules.Function;
import ru.levin.modules.FunctionAnnotation;
import ru.levin.modules.Type;
import ru.levin.modules.setting.BooleanSetting;
import ru.levin.modules.setting.MultiSetting;
import ru.levin.modules.setting.SliderSetting;
import ru.levin.util.math.GCDUtil;

import java.util.Arrays;

@FunctionAnnotation(name = "AimBot", desc = "Мгновенное плавное наведение", type = Type.Combat)
public class AimBot extends Function {
    
    public BooleanSetting player = new BooleanSetting("Игроки", true);
    public BooleanSetting golyPlayer = new BooleanSetting("Голые", true);
    public BooleanSetting mobs = new BooleanSetting("Мобы", false);
    public BooleanSetting friend = new BooleanSetting("Друзья", false);
    public MultiSetting targets = new MultiSetting(
            "Цели",
            Arrays.asList("Игроки", "Голые", "Мобы", "Друзья"),
            new String[]{"Игроки", "Голые", "Мобы", "Друзья"}
    );

    public SliderSetting speed = new SliderSetting("Скорость", 1.0f, 0.8f, 1.0f, 0.01f);
    public SliderSetting delay = new SliderSetting("Задержка", 0, 0, 3, 1);
    public SliderSetting distance = new SliderSetting("Дистанция", 3.2f, 1.5f, 6f, 0.1f);

    public BooleanSetting eatPause = new BooleanSetting("Пауза при еде", true);
    public BooleanSetting blockPause = new BooleanSetting("Пауза при блоке", true);
    public BooleanSetting onlyAttack = new BooleanSetting("Только при атаке", false);
    public BooleanSetting raycast = new BooleanSetting("Проверять видимость", true);
    public BooleanSetting silent = new BooleanSetting("Silent", false);
    
    private static LivingEntity target;
    private int delayCounter = 0;
    private boolean wasAttacking = false;

    public AimBot() {
        addSettings(targets, speed, delay, distance, eatPause, blockPause, onlyAttack, raycast, silent);
    }

    @Override
    public void onEvent(Event event) {
        if (!(event instanceof EventUpdate)) return;
        if (mc.player == null || mc.world == null) return;

        // Проверка пауз
        if (eatPause.get() && mc.player.isUsingItem()) return;
        if (blockPause.get() && mc.player.isBlocking()) return;

        if (onlyAttack.get()) {
            boolean attacking = mc.options.attackKey.isPressed();
            if (!attacking) {
                wasAttacking = false;
                return;
            }
            if (!wasAttacking && attacking) {
                delayCounter = 0;
                wasAttacking = true;
            }
        }
        
        if (target == null || !isValidTarget(target)) {
            updateTarget();
        }

        if (target == null) return;
        
        if (delayCounter < delay.get().intValue()) {
            delayCounter++;
            return;
        }
        delayCounter = 0;

        aimDirect(target);
    }

    private void aimDirect(LivingEntity entity) {
    
        Vec3d targetPos = entity.getPos().add(0, entity.getHeight() * 0.5, 0);
        Vec3d eyePos = mc.player.getEyePos();

        double dx = targetPos.x - eyePos.x;
        double dy = targetPos.y - eyePos.y;
        double dz = targetPos.z - eyePos.z;

        float targetYaw = (float) Math.toDegrees(Math.atan2(-dx, dz));
        float targetPitch = (float) Math.toDegrees(Math.atan2(-dy, Math.sqrt(dx * dx + dz * dz)));

        float currentYaw = mc.player.getYaw();
        float currentPitch = mc.player.getPitch();

        float yawDelta = MathHelper.wrapDegrees(targetYaw - currentYaw);
        float pitchDelta = targetPitch - currentPitch;

        float spd = speed.get().floatValue();

        float newYaw = currentYaw + yawDelta * spd;
        float newPitch = currentPitch + pitchDelta * spd;

        newPitch = MathHelper.clamp(newPitch, -90f, 90f);


        if (silent.get()) {
            mc.player.setYaw(newYaw);
            mc.player.setPitch(newPitch);
        } else {
            mc.player.setYaw(GCDUtil.applyGCD(newYaw, currentYaw));
            mc.player.setPitch(GCDUtil.applyGCD(newPitch, currentPitch));
        }
    }

    private void updateTarget() {
        LivingEntity bestTarget = null;
        double bestDistance = Double.MAX_VALUE;

        for (Entity entity : mc.world.getEntities()) {
            if (!(entity instanceof LivingEntity living)) continue;
            if (!isValidTarget(living)) continue;

            double dist = mc.player.distanceTo(living);
            if (dist < bestDistance) {
                bestDistance = dist;
                bestTarget = living;
            }
        }
        target = bestTarget;
    }

    private boolean isValidTarget(LivingEntity entity) {
        if (entity == mc.player) return false;
        if (mc.player.distanceTo(entity) > distance.get().floatValue()) return false;

        // Проверка видимости
        if (raycast.get() && !mc.player.canSee(entity)) return false;

        // Фильтрация по типу
        if (entity instanceof PlayerEntity player) {
            if (!targets.get("Игроки")) return false;
            if (!targets.get("Друзья") && Manager.FRIEND_MANAGER.isFriend(player.getName().getString())) return false;
            if (!targets.get("Голые") && player.getArmorVisibility() <= 0) return false;
            if (player.isCreative()) return false;
        } else if (entity instanceof Monster || entity instanceof AnimalEntity || entity instanceof VillagerEntity) {
            if (!targets.get("Мобы")) return false;
        }

        // Исключения
        if (entity instanceof ArmorStandEntity) return false;
        if (entity.isInvulnerable()) return false;
        if (!entity.isAlive()) return false;

        return true;
    }

    @Override
    protected void onDisable() {
        target = null;
        delayCounter = 0;
        wasAttacking = false;
        super.onDisable();
    }
}
